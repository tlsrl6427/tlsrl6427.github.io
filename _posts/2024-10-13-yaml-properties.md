---
title: 멀티 모듈 적용기 
categories: [Spring]
tags: [yml, yaml, multi module]
---

### 멀티 모듈

 멀티 모듈을 처음으로 사용해봤다. 이유는 아주 간단하게 보기에 헷갈리기 때문이었다. 사이드 프로젝트를 두개정도 해보며 많은 기능이 없어도 파일들이 엄청 많이 생기고 정리하려면 날을 잡아야될 정도란걸 알게 됐다. 또 각각 다른 port에서 실행되는 jar파일이 있기 때문에 각자 빌드가 필요했다. 그래서 될 수 있으면 만들때부터 구조를 맞춰 작성하려고 멀티 모듈을 사용했다. common, api, batch의 모듈로 나누었다.

### 설정 파일(build.gradle, application.properties)

- build.gradle

 common, core 등의 이름의 모듈에 공통 기능을 넣곤하는 것처럼 build.gradle도 공통 라이브러리 관리와 모듈마다의 라이브러리 관리를 할 수 있다.
 
 ![img]()

- application.properties

 build.gradle을 생각보다 쉽게 해치웠기 때문에 application.properties도 똑같을 거라 생각했다. common 모듈의 application.properties에 공통 설정을 하고 나머지 모듈은 각자의 설정을 한 파일을 가졌다. 근데? 안된다?
 
 ![img]()


### YamlPropertySourceFactory

 찾아보니 공통의 application.properties를 가지려면 따로 설정을 해주어야했다. 이유는 스프링 깃허브에 나와있는데 설정 파일을 탐색할 때 자기 클래스 내에서만 하기 때문이다.
 
 ![img]()

 그래서 PropertiesConfig를 만들어 경로를 지정해주고 yaml형식의 파일이기 때문에 YamlPropertySourceFactory를 구현해 읽을 수 있도록 해야한다
  ![img]()


### Github Actions Secret Variable

 해치운줄 알았는데 뜻밖의 오류를 마주했다.
  ![img]()


