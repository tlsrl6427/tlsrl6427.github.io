---
title: "Enum Custom 예외처리하기"
categories: [Spring]
tags: [enum, exception, converter, aop]
---

### API에서 Enum을 사용할때의 문제점
 간단하게 Enum들(Sports, Colors)와 EnumRequest를 만들고 EnumController를 통해 API 통신하는 프로젝트를 만들었다. 입력값은 URL에 적어서 가며 파라미터들을 EnumRequest로 한꺼번에 받기 위해 @ModelAttribute를 사용하였다. 그런 후 Enum값과 다른 값을 줬을 때 어떤 에러가 나올까?<br>
Sports의 값 중 FootBall -> football로 바꾸고 요청을 해보았다.
![img1](/assets/img/2024-08-16-enum-custom/img1.png)
콘솔에서는 'football'이란 단어에 대해 String 타입을 Sports 타입으로 변경하는데에 실패했다고 나온다.
![img2](/assets/img/2024-08-16-enum-custom/img2.png)
그리고 포스트맨(클라이언트 입장)에서는 어떤 오류가 생겼는지 알 수 없다. 때문에 Enum Convert 에러가 떠도 사용자 입장에서 알 수 있도록 바꿔볼 생각이다.

### 해결방법

이걸 해결하기 위해서는 크게 3가지가 필요했다.<br>
<br>
1. StringToEnumConverter + EnumMethod
2. @NotNull(@Valid)
3. GlobalExceptionHandler(AOP)
<br>
&nbsp;첫 번째로 Request가 도착했을 때 모든 글자는 String에서 StringToXXXConverter를 통해 알맞는 타입으로 바뀐다. 1번의 기능은 String이 Enum으로 바뀔때 우리가 의도한대로 바뀌도록 하는 것이다. String이 Enum으로 바뀔때는 기본적으로 Enum.valueOf() 함수를 통해 바뀌기 때문에 글자가 모두 일치할때만 통과되고 아니면 MethodArgumentNotValidException이 뜬다. 때문에 Custom Converter를 만들어 대소문자 상관없이 통과되고, 통과되지 않을 땐 예외로 넘어가는 것이 아닌 null이 반환되게 만들었다.<br>
&nbsp;두 번째는 Request 객체에 @NotNull을 붙이고, Controller에서 인자를 받는 곳에 @Valid를 붙이는 것이다. 그리고 괄호안에 메세지를 넣으면 (ex. @NotNull(message = "null 입니다")) null일 경우 message의 내용이 예외객체에 들어간다. 이렇게 되면 1번에서 함수를 정상적으로 통과할때는 알맞는 Enum Value가 매칭되고 비정상적인 모든 경우에는 null로 변해 @NotNull의 메세지 내용을 담게 되는 것이다.<br>
&nbsp;세 번째는 2번 내용을 담은 예외를 사용자에게 보여주는 것이다. 사실 이게 제일 중요한 거긴하다. 이것만 있어도 어떤 에러가 나왔는지 사용자가 볼 수는 있기 때문이다. @RestControllerAdvice를 선언한 클래스에서 특정 예외에 @ExceptionHandler를 걸어주면 그 예외가 터질때는 이쪽으로 넘어와서 처리가 되게 된다. 그때 내용에 에러의 내용을 ResponseEntity에 넣으면 사용자가 확인할 수 있다. 지금은 MethodArgumentNotValidException만 예외를 걸면 되어서 이 예외에 대한 내용만 넣었지만 상위 객체인 BindException이나 그 상위인 Exception을 넣으면 훨씬 넒은 범위의 에러를 한꺼번에 처리할 수도 있다. 나중에 언젠간 보겠지만 @ModelAttribute가 아닌 @RequestBody를 사용할 때는 다른 에러로 뜨기 때문에 다른 메소드를 작성해놓아야한다.

<결과>
![img3](/assets/img/2024-08-16-enum-custom/img3.png)

### 더 좋은 방법?

&nbsp;이 방법은 이해하기 쉬웠고 결과물도 만족스러웠지만 딱 한가지, 중복이 너무 많았다. 하나의 Enum Type당 StringToXXXConverter를 만들고 그안에서 쓰일 메소드를 미리 Enum에서 만들어 놓고 있어야한다. 그리고 그 메소드 내용은 모두 동일했다. 두세개까지는 괜찮았는데 웬만한 작은 프로젝트에도 7~8개의 Enum은 들어갔고 7~8번의 복붙을 하는 과정이 지루하기도 했고 붙여놓고 이름바꾸는데 틀리는 경우도 있었다. 이 중복을 줄일 수 없을까 해서 인터넷을 열심히 뒤져보았다. 그 와중에 파싱 메소드에 @JsonCreator를 붙이는 방법, Enum이 들어간 객체를 위한 커스텀 Enum 어노테이션을 만드는 방법, ConverterFactory를 사용하는 방법이 있었다.<br>
<br>
- @JsonCreator?
&nbsp;아시는 분은 아시겠지만 이 방법은 틀렸다. 이건 @RequestBody에서만 동작한다. 원리는 Http에서 body로 오는 json을 파싱할때 @JsonCreator를 선언해놓으면 원래 메소드 대신 @JsonCreator하단에 적힌 메소드로 대신 파싱해주는 것이었다. 이 방법은 Converter를 따로 만들 필요없이 파싱 메소드 위에 선언만 하면 되기때문에 중복을 반으로 줄여주는 효과가 있다. 그래도 중복 자체는 있기 때문에 @RequestBody를 사용할때 중복을 아예 없애는 방법도 생각해봐야될 것 같다.<br>
<br>
&nbsp; 커스텀 Enum 어노테이션은 내 기준에서는 조금 어려워서 패스했고 ConverterFactory 내용을 살펴보았다. ConverterFactory는 String을 Enum으로 바꿀
### 채택한 방식
