---
title: "Spring Batch Job Parameter로 List Type 쓰기"
category: "Spring"
tags: "spring, batch, job parameter, list"
---

## 프로젝트 상황

&nbsp;현재 하고 있는 프로젝트는 op.gg와 lol.ps 같은 리그오브레전드 게임의 데이터를 제공하는 사이트이다. 라이엇 공식 API에서 데이터를 받아오는 것을 Spring Batch로 돌리고 있는데 이때 동적으로 넣을 Job Parameter가 필요하게 됐다.   
<br>
&nbsp;필요한 항목은 티어(tier), 점수(leaguePoints), 오늘 날짜(curDate)이다.

## 현재 버전(5.x) 지원

 Spring Batch 5.0 전까지는 Date, Double, Long, String만 받을 수 있기 때문에 LocalDate와 LocalDateTime을 처리하기 위해서 String을 LocalXXX로 변환하는 글도 심심치 않게 볼 수 있었다. 하지만 5.0부터는 LocalDate와 LocalDateTime이 추가되어 형식만 맞춘다면 쉽게 받을 수 있도록 바뀌었다. 
 
## List 사용하기

### Default notation

### Extended notation(JSON)

### Json 직렬화

### Class 사용과 그 밖에 ..
