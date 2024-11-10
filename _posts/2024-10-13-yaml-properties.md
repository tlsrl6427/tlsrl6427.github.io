---
title: Github Secrets 쓸 때 주의사항(with 특수문자)
categories: [Spring]
tags: [yml, yaml, multi module, github secrets]
---

## 멀티 모듈

&nbsp;멀티 모듈을 처음으로 사용해봤다. 이유는 아주 간단하게 보기에 헷갈리기 때문이었다. 사이드 프로젝트를 두 개정도 해보며 많은 기능이 없어도 파일들이 엄청 많이 생기고 정리하려면 날을 잡아야될 정도란걸 알게 됐다. 또 이번 프로젝트는 각각 다른 port에서 실행되는 jar파일이 있었기 때문에 각자 빌드가 필요했다. 그래서 될 수 있으면 만들때부터 구조를 맞춰 작성하려고 멀티 모듈을 사용했고 common, api, batch의 모듈로 나누었다.   

&nbsp;공통 build.gradle과 application.yml를 만들었다. application.yml 같은 경우엔 같이 쓰기 위해서는 직접 지정해줘야해서 PropertiesConfig와 YamlPropertySourceFactory를 만들어주기도 했다.

<br>

## Github Actions Secret Variable(with Special Characters)

### Docker logs에 찍힌 Spring 오류

 해치운줄 알았는데 뜻밖의 오류를 마주했다.
 
 ![img3](/assets/img/2024-10-13-yaml-properties/img3.png)

&nbsp;뜬금없이 또 데이터베이스에서 비밀번호가 일치하지 않는다는 것이다. 비밀번호를 메모장에 쳐놓고 복붙해서 그럴리가 없는데.. 포트도 보고, 유저 접근 권한도 봤는데 이상없었다. 그러던 중 눈에 띄는 것이 있었다. using password...No? 분명 비밀번호는 yaml에 있을텐데..? 

![img4](/assets/img/2024-10-13-yaml-properties/img4.png)

&nbsp;yaml에 있는 비밀번호를 일부러 틀리게 해서 실행해보았다. 그랬더니 이번엔 using password가 yes로 바뀌었다. 이렇게되니 yaml에서 인식 못하는 문자가 포함되었다는 생각이 들었고, 비밀번호를 바꿨더니 정상적으로 되었다.

<br>

### 원인

&nbsp;먼저 yaml에 사용하면 안되는 문자들을 찾아보았다.

![img5](/assets/img/2024-10-13-yaml-properties/img5.png)

&nbsp;이상하다... 없다. 원래 비밀번호는 $로 시작하고 영문자, 숫자가 섞인 8자 이상의 비밀번호였다. 어찌됐든 yaml에서 문제가 발생한 것은 맞으니 그럼 Github Actions에서 빌드하는 과정에 문제가 있나..? 하고 추측할 수 밖에 없었고 그에 대해 찾아보기로 했다. 

&nbsp;이 과정을 멘토님께 말씀드리니 [Github Community 글](https://github.com/orgs/community/discussions/25651)을 찾아주셨다.

![img6](/assets/img/2024-10-13-yaml-properties/img6.png)

&nbsp;Github Actions를 실행할때 Github Secret에서 값을 고대로 가져온다. 이때 Github Actions는 ubuntu환경에서 실행돼서 $로 시작하는 문구는 "$변수명"으로 인식해 당연히 저 이름의 변수명이 없으니 빈 값으로 치는 것이었다. 그렇다면 linux 계열에서 쓰는 $, &, ( 등의 문자들은 다 사용할때 주의해야한다는 것이다. 음... 그렇군 그럼 최대한 사용하지 말아야겠다.

<br>
<br>

### 해결방법

&nbsp;정말 사용하고 싶지 않았는데 어쩔 수 없이 사용해야만 하는 상황이 왔다. Spring Batch를 써본 분이라면 이 문구를 알고 있을 것이다.

```yaml
spring:
  batch:
    job:
      name: ${job.name:NONE}
```

&nbsp;Spring Batch 3부터는 multiple job을 돌리는 것을 막았다. 2개 이상의 job이 있으면 실행할 job을 명시하지 않을시 이러한 오류가 뜬다.

![img7](/assets/img/2024-10-13-yaml-properties/img7.png)

&nbsp;따라서 job.name에 사용할 job을 명시해야하고, 만약 jar파일을 실행할 때 동적으로 Program argument에 사용할 job을 넣고 싶으면 ${job.name:NONE}를 추가해야한다( ${job.name:NONE}없이 Program argument에 값을 추가해도 저 오류가 뜬다 ). 

근데... $가 있네...?

<br>
<br>

1. \ 넣기

![img8](/assets/img/2024-10-13-yaml-properties/img8.png)

&nbsp;[스택오버플로우 글](https://stackoverflow.com/questions/77605186/special-characters-in-github-actions-workflow-secret-are-not-being-preserved)에서 escape 문자를 사용하라고 되어있다. 흠.. 근데 Intellij에서 보기에 별로 안좋을 것 같고 Github Secret에 추가할때만 \를 다 넣는 것도 번거로웠다. 일단 더 찾아봐야겠다.

<br>

2. ~~따옴표로 묶기~~

![img9](/assets/img/2024-10-13-yaml-properties/img9.png)

&nbsp;[다른 블로그 글](https://www.ssw.com.au/rules/handle-special-characters-on-github/)에서는 따옴표(")로 묶으라고 되어있다. 하지만 바로 문제점이 나오는데 Secret 파일에 "가 있을 경우 또 같은 문제가 발생한다는 것이다. 근데 이거 아니고도 해봤는데 나는 그냥 안됐다. 계속 인식을 안했다.

<br>

3. Secret을 Base64로 인코딩하기
 
 ![img10](/assets/img/2024-10-13-yaml-properties/img10.png)

&nbsp;아예 특수문자가 걸릴일이 없게 yaml파일을 Base64로 인코딩한 후, workflow 파일에서 빌드시 디코딩하는 방법이다. 신박한 방법인 것 같다. 생각해보니 예전에 AWS를 사용할 때 pem 파일을 Base64로 인코딩한 값을 key로 넣어 접속했던 기억이 난다. 똑같이 하면 될 것 같은데 더 좋은 방법이 있나 더 찾아봤다.

<br>

4. env에 선언하기

 ![img11](/assets/img/2024-10-13-yaml-properties/img11.png)

&nbsp;위의 멘토님이 보내준 글에 있던 방법이다. 여기서 나와있길 따옴표로 감싸져있는거 다 풀어진다고 한다. 어쩐지 안되더라. 

```yaml
 # SECRET_YML 파일 생성
    - name: Make application-secret.yml
      run: |
        cd ./common/src/main/resources
        touch ./application-common.yaml
        echo "\${{ secrets.COMMON_YML }}" > ./application-common.yaml

        cd /home/runner/work/LOLTMI/LOLTMI
        cd ./batch/src/main/resources
        touch ./application-secret.yml
        echo "${SECRET_YML_BATCH}" > ./application-secret.yml
      env:
        SECRET_YML_BATCH: \${{ secrets.SECRET_YML_BATCH }} // env에 선언
```

&nbsp;workflow에서 run 할때 따로 변수를 선언하고 거기다 초기화를 하면 된다. 나는 이 방법이 먹혔고 제일 낫다고 생각해 이렇게 쓰기로 했다. env에 대해선 [Github Docs](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#env)에 나와있는데 뭐 별거없다.

<br>

### 끝?

 &nbsp; 끝이다. 정말 우연히 데이터베이스 비밀번호에 $를 쓰는 바람에 많~이 돌아왔다. 이런게 막상 부딪히면 어떻게 해결할지 모를때가 많은데 이번엔 해답을 찾아서 다행이다. 혹시 이 글로 Github Secret에 $를 쓰면 안되는지 처음 알았다면 기억해두자,,
