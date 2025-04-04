---
title: "Gradle 빌드시 encoding 문제"
category: "Java"
tags: ["encoding", "gradle"]
---

&nbsp;Rest Docs를 만들기 위해 빌드를 하는 도중 일어난 오류이다. 2023.11.17에 티스토리에 썼던 글(https://ignihs.tistory.com/15)

## 문제의 발단
---

 ![img1](/assets/img/‎2025-04-03-gradle-encoding/img1.png)

&nbsp; 팀원이 Rest docs를 만들기 위해 ./gradlew build를 하는 도중 오류가 떴다. 웬만하면 각자 해결했겠지만 이번에는 집단지성이 필요했는데 인텔리제이에서는 잘 돌아갔는데 gradle로 빌드를 하면 오류가 떴기 때문이다. 심지어 오류 메세지도 아리송하다.   

&nbsp;400 에러는 Bad Request, 즉 서버에서 요구하는 Request에 맞지 않는 요청을 보냈다는 뜻이다. Postman으로 요청을 했다면 모를까 테스트에서는 직접 객체를 만들어 MockMvc.perform안에 넣었기 때문에 틀릴리가 없을텐데...? 일단 이 오류메세지 만으로는 알 수 없으니 인텔리제이에서 직접 확인해보도록 하자.

## 해결과정
---

### 1. 인텔리제이에서 테스트 속성을 gradle로 하고 실행하기

![img2](/assets/img/‎2025-04-03-gradle-encoding/img2.png)

&nbsp;프로젝트를 처음 시작할때 Build and run 속성을 Default인 Gradle로 해놓으면 느리기 때문에 두 속성을 "IntelliJ IDEA"로 바꿔놓은 경험이 있을 것이다. 나는 인프런의 김영한님 강의를 들으면서 어느순간 바꿔놓은 것 같다. 그리고 까먹고 있었다가 이 오류를 계기로 기억이 나서 바꿔보았다.

### 2. 테스트 돌리고 로그확인

![img3](/assets/img/‎2025-04-03-gradle-encoding/img3.png)

&nbsp;아래쪽 오류로그에서 오류가 나는 곳으로 이동했다. 이때 런타임 로그를 보면 마지막에 WARN이 뜬 것을 볼 수 있다.

![img4](/assets/img/‎2025-04-03-gradle-encoding/img4.png)

&nbsp;request 객체를 ObjectMapper를 이용해 바꾸는 과정에서 에러가 나는 것 같다. 

### 3. 인코딩 오류

![img5](/assets/img/‎2025-04-03-gradle-encoding/img5.png)

&nbsp;음.. 인코딩 어쩌구저쩌구 뭐라그러고 UTF-8 인코딩으로 읽으려고 하는데 알 수 없는 문장이라고 한다. 그렇다면 request가 UTF-8 형식으로 잘 바뀌지 않았나보다. 

## 해결방법
---

&nbsp;인코딩을 적용해주면 되는 문제라 여러가지로 풀 수 있었다.

### 1. getBytes()안에 UTF-8 명시

![img6](/assets/img/‎2025-04-03-gradle-encoding/img6.png)

&nbsp;ObjectMapper쪽 문제인걸 인지하고 함수들을 살펴보다가 getBytes에 인자를 줄 수 있다는 것을 알게 되었다.

![img7](/assets/img/‎2025-04-03-gradle-encoding/img7.png)

&nbsp;java.nio.charset에 있는 클래스를 넣을 수 있다는 것이다. 저 패키지안에 있는 클래스들 중 StandardCharsets의 UTF-8 속성을 추가해주니 통과가 되었다.

### 2. writeValueAsBytes로 바꾸기

![img8](/assets/img/‎2025-04-03-gradle-encoding/img8.png)

&nbsp;writeValueAsString.getBytes() -> writeValueAsBytes로 바꾸니 통과되었다.

![img9](/assets/img/‎2025-04-03-gradle-encoding/img9.png)

&nbsp;writeValueAsBytes에 Ctrl를 누른 상태로 마우스를 클릭하면 함수 안의 내용이 나온다. 보면 JsonEncoding.UTF8로 기본설정을 하고 시작하는 것을 볼 수 있다.

### 3. gradle.properties 추가

![img10](/assets/img/‎2025-04-03-gradle-encoding/img10.png)

&nbsp;gradle.properties 파일을 만들어 안에 인코딩 정보를 추가하였다.

![img11](/assets/img/‎2025-04-03-gradle-encoding/img11.png)

&nbsp;그랬더니 아까 안됐던 writeAsValueAsString().getBytes()가 통과되는 것을 확인할 수 있다.

## 그렇다면 왜 이런 문제가 일어났을까?
---

&nbsp;사실 여기가 메인이다. 이 오류가 일어났을 당시엔 로그를 보고 구글링을 통해 비교적 쉽게 오류를 고치고 밀린 코딩을 했었다. 그리고 돌이켜 생각해보니 평소엔 프로젝트하면서 많은 테스트를 작성했고 이런 인코딩 문제가 없었는데 갑자기 왜 이런 오류가 났을까 궁금했고, 그 이유를 찾아보기로 했다. 

### 1. getBytes()의 default encoding

![img12](/assets/img/‎2025-04-03-gradle-encoding/img12.png)

&nbsp;보통 테스트할때 이런식으로 writeValueAsString까지만 한다.

![img13](/assets/img/‎2025-04-03-gradle-encoding/img13.png)

&nbsp;하지만 이 테스트는 사진도 같이 받기때문에 Multipartfile 형식의 데이터가 필요했고, 그 요구사항이 byte[]여서 getBytes()를 써야 했던 것이다.

![img14](/assets/img/‎2025-04-03-gradle-encoding/img14.png)

&nbsp;위에서도 봤던 getBytes()의 설명을 다시 읽어보면 플랫폼의 기본 문자열 집합을 사용한다고 나와있다. 지금이야 gradle.properties에 UTF-8이라고 명시해놨지만 원래는 무엇이었을까?

### 2. platform에 따른 인코딩

&nbsp;일단 platform이란 무엇일까? 말그대로 테스트를 돌리는 주체이다. 아까 Intellij Settings에서 "Run tests Using" 속성을 gradle로 했던것을 기억할 것이다. gradle.properties를 놔둔 상태에서 이 속성을 "Intellij IDEA"로 바꾸고 File Encodings을 모두 default(나는 windows를 사용하기 때문에 x-windows-949)로 바꿔보자.   

&nbsp;그렇게 하면 테스트는 실패하고, 다시 "Gradle"로 바꾸면 성공한다. Gradle은 gradle.properties가 있기 때문이다. 길어질까봐 사진은 생략하고 어쨋든 지금은 테스트를 돌리는 주체는 뭐다? Gradle이다. 그렇다면 Gradle의 기본 인코딩을 찾아보자.

![img15](/assets/img/‎2025-04-03-gradle-encoding/img15.png)

&nbsp;Gradle Forum이라는 곳에 올라온 글의 답글이다. Gradle Employee라고 되어있어서 들어가서 살펴보니 같은 집단에 Core Dev라는 직책도 있고 연관 링크에 Gradle 공홈이 걸려있는 것으로 보니 찐 Gradle 개발자들인 것 같다.   

&nbsp;아무튼 설명을 보자면 Gradle은 JVMs platform의 인코딩을 따른다고 되어있다. 흠... 또 플랫폼?

### 3. Gradle의 기본 인코딩

![img16](/assets/img/‎2025-04-03-gradle-encoding/img16.png)


&nbsp;이번엔 스택오버플로우를 참고해보자. 이 답변에서는 JVM은 돌아가는 시스템의 문자집합을 따라간다고 한다. 결국엔 Gradle -> JVM -> OS 순으로 참조하는 것이고 즉 OS의 기본 인코딩을 따라간다는 것이다.   
&nbsp;그리고 내가 쓰는 windows의 기본 인코딩? MS949이다. 눈으로 확인할 길이 있을까?
 
![img17](/assets/img/‎2025-04-03-gradle-encoding/img17.png)

&nbsp;정말정말 친절하게도 답글을 내리다 보니 cmd창에 볼수 있는 명령어도 제시해주었다. 바로 쳐보자.

![img18](/assets/img/‎2025-04-03-gradle-encoding/img18.png)

&nbsp;file encoding을 MS949로 한다고 명시적으로 나와있다. 파고파서 결국엔 해답을 얻었다.

## 정리
---

1. Gradle로 테스트 돌릴때 인코딩 문제가 생김
2. UTF-8로 변환할 수 없다는 내용이 주된 내용
3. Gralde의 default charset은 JVM의 default charset을 따르고 JVM은 OS의 default charset을 따름
4. 따라서 현재 OS(Windows)의 default charset인 MS949를 사용하고 있어서 에러가 생긴것임
5. 인코딩 설정을 UTF-8로 바꾸니 해결됨

&nbsp;이게 끝이라고 생각할 수도 있지만 사실 근본적인 궁금증이 하나 더 있긴하다. 오류 메세지를 보면 HttpMessageConverter에서 UTF-8로 변환할때 문제가 생긴 것 같은데 이 변환 과정에 대한 이해도 부족한 것이다.. 그래서 궁금해서 찾아봤는데 그 내용을 쓰려면 이만한 글을 하나 더 써야될 것 같다.   

&nbsp;요약해서 말하자면 HttpMessageConverter는 @ResponseBody같은게 붙은 데이터를 자동으로 바꿔주는 것인데 String은 UTF-8로 인코딩해준다고 한다. 근데 테스트에서 writeValueAsString으로 바뀐 request 객체를 getBytes()를 거치며 MS949형식의 Bytes로 바꿔버렸고 그걸 읽으려고 보니까 인코딩 형식이 서로 맞지않아 읽지 못하는 것이다.   

&nbsp;일단은 이렇게 마무리한다. 저 내용도 중요한거라 공부하고 있는데 이 글과는 거리가 조금 있기도 하고 너무 길어지기도 해서..

### 참고한 사이트

https://discuss.gradle.org/t/is-there-a-way-to-tell-gradle-to-read-gradle-build-scripts-using-a-specified-encoding/7535
https://stackoverflow.com/questions/1006276/what-is-the-default-encoding-of-the-jvm
