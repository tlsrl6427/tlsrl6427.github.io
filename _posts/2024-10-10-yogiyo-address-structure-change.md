---
title: 요기요 주소 체제 바꾸기
categories: [Database]
tags: [query tunning, mysql]
---

 「요기요」 앱을 클론코딩하는 프로젝트를 진행하였다. 보는대로 따라만드는 거라 큰 어려움은 없었다. 점주 입장에서 상점을 등록하고 고객 입장에서 상점을 조회하는, 게시판의 확장 버전 정도? 그렇게 설계와 구현이 끝나고 테스트를 하다보니 여러가지 문제점이 발생했다. 그 중 대표적인 것이 상점 리스트를 조회하는 페이지가 눈에 띄게 느려진 것이다. 처음엔 어떻게든 바꾸고 바꿔서 온몸 비틀기하다가 결국에는 한계를 보고 설계를 다시 생각하는 상황까지 왔다. 처음 시작부터 해결하기까지의 과정을 살펴보자.

## Shop 테이블 설계
<br>
 <table>
   <tr>
     <td>기능</td>
     <td>내용</td>
   </tr>
   <tr>
     <td>정렬</td>
     <td>주문 많은 순, 리뷰 많은 순, 별점 높은 순, 거리순</td>
   </tr>
   <tr>
     <td>필터링</td>
     <td>카테고리, 최소 주문 금액, 배달금액</td>
   </tr>
 </table>
 <br>

- ERD

- 조회 쿼리
 
```sql
select *, st_distance_sphere(point(curLongitude, curLatitude), point(s.longitude, s.atitude)) as distance
from shop s
join category c on s.category_id = c.id // category 테이블과 조인
where (
    c.name = ??? // 카테고리
    and s.least_order_price < ??? // 최소주문금액 
    and s.min_delivery_price < ??? // 최소배달금액
    and distance < 10000 // 반경 10km 이내 상점만 조회
)
order by SortOption(orderNum, reviewNum, totalScore, distance)
```
<br>

- 고려사항

1. 한 가게에 category가 여러개일수도 있기 때문에 category 테이블과 category:shop 테이블을 만들었다
2. order과 review는 각각 테이블이 있고 FK로 shop_id를 가지고 있는 ManyToOne 연관관계이다. shop_id를 기준으로 조인하고 count() 쿼리로 orderNum과 reviewNum을 가져올 수 있는데 그렇게 하면 두 가지 문제점이 있다
   
   2-1. 성능이 안좋다
     - order 테이블과 review 테이블도 각각이 로우가 많기 때문에 조회를 할때마다 count를 날리는 것은 안해봐도 느릴 것 같았다.
       
   2-2. 테이블이 뻥튀기될 수 있다
     - 보통 쿼리 하나에 해결하려고 모두 조인해놓고 시작할 때가 많은데, 이미 [shop 테이블:category 테이블] 조인을 했기 때문에
       [shop 테이블:order 테이블]도 같이 조인해버리면 (category 개수 * order 개수)개의 행이 나와버린다.
        이것을 피하는 방법은 from절에서 count()를 먼저 실행하기, 애플리케이션에서 두번이상 쿼리작업하는 등이 있다.

여러가지를 고려해보다가 shop 테이블에 orderNum과 reviewNum을 만들고 order과 review가 생성될 때마다 +1 해주도록 바꿨다. 제대로 만드려면 단일 책임 원칙을 따라 advice를 만들어 createOrder, createReview와 별개로 orderNum과 reviewNum을 +1하는 로직을 만들어야 됐을 것 같은데 귀찮아서 createXXX에 다 때려넣었다. [저스틴 비버문제](https://bezzang2.tistory.com/m/145)를 참고했다.

3. 그냥은 st_distance_sphere 함수를 쓸 수 없다. JPA에서 기본으로 제공하는 MariaDBDialect에는 이 함수를 제공하지 않기 때문이다. 그래서 따로 MariaDBDialectConfig를 만들어 등록해줬다.

```java
public class MariaDBDialectConfig extends MariaDBDialect {

    public MariaDBDialectConfig(){
        super();
        registerFunction(
                "st_distance_sphere",
                new StandardSQLFunction("st_distance_sphere", StandardBasicTypes.DOUBLE));
    }
}
```
```yaml
spring:
  jpa:
    database-platform: toy.yogiyo.common.config.MariaDBDialectConfig
```

4. 최소주문금액과 최소배달금액
   
일단 기능이 되는지 보는 것이어서 상대적으로 세세한 부분인 최소주문금액과 배달금액은 컬럼 하나로 퉁치고 넘어갔었다. 원래대로라면 shop 테이블에 있는 것이 아닌 shop_id를 FK로 가진 테이블로 따로 독립이 되어야한다. 거리마다  [0, 1000, 3000, 10000] 이런식으로 여러개이기 때문이다.

ex) 개선 후 테이블
<table>
  <tr>
    <td></td>
    <td>주문금액</td>
    <td>배달금액</td>
    <td>shop_id</td>
  </tr>
  <tr>
    <td>1</td>
    <td>50,000원 이상 주문시</td>
    <td>1,000원</td>
    <td>1</td>
  </tr>
  <tr>
    <td>2</td>
    <td>30,000원 이상 주문시</td>
    <td>3,000원</td>
    <td>1</td>
  </tr>
  <tr>
    <td>3</td>
    <td>12,000원 이상 주문시</td>
    <td>4,000원</td>
    <td>1</td>
  </tr>
</table>

 테이블을 만들고 조인에 추가하는 건 그렇다치고 거리를 고려하지 못한다는게 더 큰 문제였다. 똑같은 주문금액이어도 1km와 3km를 배달할때 배달비가 차이가 나야할텐데 그런 것까지는 고려할 수 없었기 때문이다.
 
### 데이터가 쌓여가면서 생기는 문제

 더미 데이터가 10만건을 넘기기 시작했을 때의 일이다. 점점.. 느려진다. 어떤 정렬을 하던 필터링을 하던 5초 정도나 걸려서야 조회가 됐다. 어디서 문제가 일어나는지는 뻔했다. 데이터베이스였다. 애플리케이션은 DTO로 감싸서 보내주는 것만 수행했고 데이터베이스는 Amazon RDS Freetier(MariaDB)를 사용하기 때문에 메모리가 1GB밖에 안돼서 사실 언제 터지나 궁금하긴했다. 원인은 st_distance_sphere 함수였다. 쿼리를 날릴 때 주위의 상점만 조회하기 위해 모든 로우에 대해 st_distance_sphere를 실행하기 때문에 시간이 오래걸리는 것이었다. 애플리케이션에서 거리계산하는 부분을 수행해볼까도 생각해봤는데 어차피 애플리케이션도 똑같이 Amazon EC2 Freetier라 별다를게 없을 것 같아 패스하고, 쿼리를 어떻게 건드려보기로 했다. 

### 쿼리튜닝

 문제 원인이 st_distance_sphere 함수였기 때문에 최대한 이 함수를 덜 쓰는 방향으로 생각했다. 처음부터 모든 거리를 계산하지 말고 일단 필터링으로 거를건 먼저 거르고, 주변 10km내에 있는 상점을 가져오기로 했다.

### 주소 구조 변경

### 결과

