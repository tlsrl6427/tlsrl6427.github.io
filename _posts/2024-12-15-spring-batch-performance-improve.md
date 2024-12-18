---
title: "Spring Batch 성능 개선기"
category: "Spring"
tags: ["spring batch", "jdbc", "batch insert"]
published: false
---

## 현재 상황

  API 서버는 통계 테이블 생성을 통해 해결을 보았지만, Jenkins 서버는 여전히 들어갈 수조차 없다. 이유는 아마 Jenkins 서버 안에서 돌아가는 Spring Batch의 I/O가 많아서인 것 같다. 콘솔로 들어가 vmstat으로 모니터링 해보니 wa(I/O 동안 Cpu가 노는 정도)가 100%에 육박했다. 

  ![img1](/assets/img/2024-12-15-spring-batch-performance-improve/img1.png)
  
  아니 아무리 Cpu가 바빠도 Jenkins 웹 사이트에 못 들어가는게 말이 안되는게 1개의 Cpu여도 Context Switching을 하며 다른 작업도 할 수 있는게 아닌가?? 하는 생각이 들지만 여전히 그.. 뭐.. 잘 모르겠다. 이에 대한 해결책으로 검색도 해보고 GPT에 물어보고 몇 가지를 생각해봤다.

  1. Spring Batch의 성능 개선하기 - 성능을 개선해 I/O를 줄인다면 Cpu가 다른 작업도 할 수 있을 것이다.
  2. Non-Blocking 전환 - 그냥 얕은 지식으로 Non-Blocking 전환을 한다면 I/O 작업 동안 다른 일을 할 수 있지 않을까 싶었는데 몇 개의 블로그를 봐본 결과 안된다고 한다
  3. 프로세스 우선순위 미루기 - 어차피 배치 작업이 급한 작업은 아니라 모니터링이나 웹 사이트 접속 등의 다른 프로세스에게 우선순위를 주는 건 어떨까 싶었다. 가볍게 nice 값을 변경해보라 해서 해봤는데 가볍게 안되길래 일단 패스했다.


<br>
<br>

## Spring Batch 개선

![img2](/assets/img/2024-12-15-spring-batch-performance-improve/img2.png)

배치 애플리케이션의 시퀀스 다이어그램이다. 이 중에서 6번, 10번이 개선의 여지가 보였다.
<br>

### ExistsBy

 매치가 데이터베이스에 이미 존재하는지 확인하기 위해 existsById 함수를 사용했다.
 
```sql
select 
  case when exists(
    select 1 
      from matches m
      where m.id = :id
  )
  then 'true'
  else 'false'
  end
```

 이 함수를 matchList에 하나씩 돌려가며 확인했다. 그런 후 false(아직 없는 매치 아이디)가 나오면 매치 데이터를 받아오는 식이었다. 이 방식은 1000개의 match가 있다면 1000번의 네트워크 I/O를 하는 것이다. 그래서 네트워크 I/O를 줄이기 위해 matchList를 한번에 보내기로 했다

```sql
select m.id
from Matches m
where m.id in (:MatchIds)
```

이렇게 하면 원래와 반대로 

### Batch Insert

## 성능 테스트
