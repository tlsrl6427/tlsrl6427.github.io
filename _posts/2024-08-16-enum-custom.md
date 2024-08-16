---
title: "Enum Custom 예외처리하기"
categories: [Spring]
tags: [enum, exception, converter, aop]
---

### API에서 Enum을 사용할때의 문제점
 간단하게 Enum들(Sports, Colors)와 EnumRequest를 만들고 EnumController를 통해 API 통신하는 프로젝트를 만들었다. 그런 후 Enum값과 다른 값을 줬을 때 어떤 에러가 나올까?<br>
Sports의 값 중 FootBall -> football로 바꾸고 요청을 해보았다.
![img1](/assets/img/2024-08-16-enum-custom/img1.png)
콘솔에서는 'football'이란 단어에 대해 String 타입을 Sports 타입으로 변경하는데에 실패했다고 나온다.
![img2](/assets/img/2024-08-16-enum-custom/img2.png)
그리고 포스트맨(클라이언트 입장)에서는 어떤 오류가 생겼는지 알 수 없다. 때문에 Enum Convert 에러가 떠도 사용자 입장에서 알 수 있도록 바꿔볼 생각이다.

### 기존 해결방식
사실은 해결해보긴 했다. 인터넷을 참고해서 @JsonCreator를 사용했다. 

### 다른 해결방식들

### 채택한 방식
