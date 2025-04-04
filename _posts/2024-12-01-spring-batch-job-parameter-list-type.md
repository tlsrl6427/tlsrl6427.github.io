---
title: "Spring Batch Job Parameter로 List Type 쓰기"
category: "Spring"
tags: "spring, batch, job parameter, list"
---

## 프로젝트 상황

&nbsp;현재 하고 있는 프로젝트는 op.gg와 lol.ps 같은 리그오브레전드 게임의 데이터를 제공하는 사이트이다. 라이엇 공식 API에서 데이터를 받아오는 것을 Spring Batch로 돌리고 있는데 이때 동적으로 넣을 Job Parameter가 필요하게 됐다.   
<br>
&nbsp;하던 중에 티어(tier) 변수에 여러 개를 넣을 상황이 생겼다. 사실 한 티어씩 여러번 배치를 돌리는 방법도 있지만 한번에 보기 쉽게 해보고도 싶었고 나중에 여러 개를 받아야하는 상황이 나올 수도 있을 것 같아서 그냥 해봤다. 하는 김에 한꺼번에 여러 변수를 넣을 수 있는지, 넣을 수 있으면 같은 타입만 가능한지 여러 개의 타입이 섞인 클래스도 가능한지 해보았다.
<br>
<br>

## 현재 버전(5.x) 지원

 Spring Batch 5.0 전까지는 Date, Double, Long, String만 받을 수 있기 때문에 LocalDate와 LocalDateTime을 처리하기 위해서 String을 LocalXXX로 변환하는 글도 심심치 않게 볼 수 있었다. 하지만 5.0부터는 LocalDate와 LocalDateTime이 추가되어 형식만 맞춘다면 쉽게 받을 수 있도록 바뀌었다. 
 그렇다면 다른 타입은 받지 못할까? 그 답은 Spring 공식 문서 [What's New in Spring Batch 5.0](https://docs.spring.io/spring-batch/docs/5.0.4/reference/html/whatsnew.html#default-job-parameter-conversion) 에서 볼 수 있다.

  ![img1](/assets/img/2024-12-01-spring-batch-job-parameter-list-type/img1.png)

  5.0부터는 standard conversion service에 converter를 추가하면 아무 타입이나 쓸 수 있다고 한다.
  <br>
<br>

## List 사용하기

  ![img2](/assets/img/2024-12-01-spring-batch-job-parameter-list-type/img2.png)
  
  기본 형식이다. 하지만 List 같은 오브젝트는 ,를 포함하기 때문에 사용하지 못한다.
  <br>

![img3](/assets/img/2024-12-01-spring-batch-job-parameter-list-type/img3.png)

  콤마가 포함된 value일 때 사용하는거라고 한다. 그러면서 JSON 형식을 소개한다. 역시 편하네 하면서 있는 예제형식대로 변수를 넣어봤다.

   ![img4](/assets/img/2024-12-01-spring-batch-job-parameter-list-type/img4.png)
<br>

### JSON 파싱 오류

 ![img5](/assets/img/2024-12-01-spring-batch-job-parameter-list-type/img5.png)

 잘 될리가 없다. 이젠 억까가 없으면 서운하다. 내용을 보니 처음부터 쌍따옴표(")가 안붙었다고 안된다고 한다. 그래서 로그에 적힌 문자열을 보니 쌍따옴표가 없어져있었다.

  ![img6](/assets/img/2024-12-01-spring-batch-job-parameter-list-type/img6.png)
<br>

### JSON 직렬화

Spring을 사용하면서 잊고 있는 사실이 있었는데 얘가 생각보다 많은 걸 편리하게 해준다는 것이었다. 그 중에 JSON 직렬화, 역직렬화가 있다. 대표적으로 쌍따옴표(")같은 문자는 이스케이프 문자가 없으면 중간에 문자열이 끝나는 것으로 인식해버려 반토막되고 에러가 뜬다. 때문에 데이터를 보내고 받을 때는 직렬화, 역직렬화가 필요한데 " 같은 문자를 이스케이프 문자로 처리해주는 것이다. Spring에서는 예를 들어, HTTP 통신을 할 때 HttpMessageConverter에서 ObjectMapper를 사용해 자동으로 직렬화, 역직렬화를 해준다.

JsonJobParametersConverter는 ObjectMapper에서 readValue를 하는데 이 함수는 들어오는 JSON 문자열이 직렬화되어있다고 생각하기 때문에 안되어있으면 위와 같은 에러가 뜨는 것이다. 

![img7](/assets/img/2024-12-01-spring-batch-job-parameter-list-type/img7.png)

때문에 직렬화를 시켜주고 넣어봤다.

![img8](/assets/img/2024-12-01-spring-batch-job-parameter-list-type/img8.png)

해치웠다. 
<br>

### Class 사용과 그 밖에 ..
