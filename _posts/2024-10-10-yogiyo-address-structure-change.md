---
title: 요기요 주소 체제 바꾸기
categories: [Database]
tags: [query tunning, mysql]
---

 「요기요」 앱을 클론코딩하는 프로젝트를 진행하였다. 보는대로 따라만드는 거라 큰 어려움은 없었다. 점주 입장에서 상점을 등록하고 고객 입장에서 상점을 조회하는, 게시판의 확장 버전 정도? 그렇게 구현이 끝나고 더미 데이터를 추가해서 실험해보는데, 여기서 문제가 발생하였다. 100만건 정도 데이터를 넣고 나니 상점 리스트를 조회하는 페이지가 눈에 띄게 느려진 것이다. 어디 부분에서 지연이 됐나했더니 데이터베이스였다. 그도 그럴 것이 데이터베이스가 무료로 쓰는 Amazon RDS(MariaDB)이어서 메모리가 1GB밖에 안됐기 때문이다. 느려도 죽지 않고 결국 조회가 되는건 신기했지만 제일 중요한 기능이라 어떻게든 개선을 해야했다.  

### Shop List 조회
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

1. 초기 쿼리
 
```sql
select *, st_distance_sphere(point(curLongitude, curLatitude), point(s.longitude, s.atitude)) as distance
from shop s
join category c on s.category_id = c.id // category 테이블과 조인
where (
    c.name = ??? // 카테고리
    and s.least_order_price < ??? // 최소주문금액 
    and s.min_delivery_price < ??? // 배달금액
    and distance < 3000 // 반경 3km 이내 상점만 조회
)
order by SortOption(orderNum, reviewNum, totalScore, distance)
```

고려사항
1. 한 가게에 category가 여러개일수도 있기 때문에 category 테이블과 category:shop 테이블을 만들었다
2. order과 review는 각각 테이블이 있고 FK로 shop_id를 가지고 있는 ManyToOne 연관관계이다. shop_id를 기준으로 조인하고 count() 쿼리로 orderNum과 reviewNum을 가져올 수 있는데 그렇게 하면 두 가지 문제점이 있다
   
   2-1. 성능이 안좋다
     - order 테이블과 review 테이블도 각각이 로우가 많기 때문에 조회를 할때마다 count를 날리는 것은 안해봐도 느릴 것 같았다.
   2-2. 테이블이 뻥튀기될 수 있다.
     - 보통 쿼리 하나에 해결하려고 모두 조인해놓고 시작할 때가 많은데, 이미 shop 테이블:category 테이블 조인을 했기 때문에
       shop 테이블:order 테이블도 같이 조인해버리면 (category 개수 * order 개수)개의 행이 나와버린다.
       이것을 피하려면 미리 order 테이블과 조인하고 count()를 실행하고 from절 join을 해야한다.

그래서 shop 테이블에 orderNum과 reviewNum을 만들고 order과 review가 생성될 때마다 +1 해주도록 바꿨다. 제대로 만드려면 단일 책임 원칙을 따라 advice를 만들어 createOrder, createReview와 별개로 orderNum과 reviewNum을 +1하는 로직을 만들어야 됐을 것 같은데 귀찮아서 createXXX에 다 때려넣었다. 

3. 그냥은 st_distance_sphere 함수를 쓸 수 없다. JPA에서 기본으로 제공하는 MariaDBDialect에는 저 함수가 없기 때문이다. 그래서 따로 MariaDBDialectConfig를 만들어 등록해줬다.
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




### 데이터가 쌓여가면서 생기는 문제
### 쿼리튜닝
### 주소 구조 변경
### 결과

