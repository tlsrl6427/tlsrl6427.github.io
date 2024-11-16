---
title: Rocky Linux 8.10 Docker Install(NCP Micro Server)
categories: [Linux]
tags: [linux, rocky linux, ncp, docker, 리눅스, 록키 리눅스, 도커]
---

&nbsp;기존에 썼던 AWS EC2 Freetier가 만료되어서 새로운 프로젝트는 NCP를 사용해보기로 했다. 계정 몇개 더파서 익숙한 AWS를 쓸까했는데 멘토링을 하는 곳에서 NCP 크레딧을 지급해줘서 그런김에 사용해봤다.

&nbps;프로젝트에서 배치를 돌려서 배치 돌리는 툴로 Jenkins를 사용했는데 전에 Docker Image로 간단히 시작했던 기억이 있어서 이번에도 Docker안에서 Jenkins를 사용하기로 했다. 근데... Docker가 안깔린다...?

### 서버 스펙과 오류

&nbsp;서버는 NCP의 무료 서버인 Micro Server( mi1-g3(vCPU 1EA, Memory 1GB) )이다. 운영체제는 따로 정할 수 없고 rocky-8.10를 쓰도록 되어있다.   
&nbsp;[블로그](https://hahahax5.tistory.com/10)를 참고해 하나하나 따라하던 중이었다. dnf config-manager로 docker repository를 추가하고 dnf repolist -v로 확인을 해봤는데 무언가 에러가 떴다.

![img1](/assets/img/2024-11-06-rocky-linux-8-10-docker-install/img1.png)

&nbsp;언제나 그랬듯이 에러 문구를 구글링해봤는데 썩 맘에 드는 답변이 나오지 않았다. 그러던 중 어떤 [도커 커뮤니티](https://forums.docker.com/t/unable-to-install-docker-on-rhel-9-2/136123)에서 직접 들어가서 봐라! 라는 문구가 있었다. 생각해보니 https로 시작하는 링크였구나 라는 걸 깨닫고 한번 들어가봤다.

![img2](/assets/img/2024-11-06-rocky-linux-8-10-docker-install/img2.png)

&nbsp;Rocky Linux가 centos계열이기 때문에 https://download.docker.com/linux/centos 로 들어가봤다. Network 탭을 들어가면 Amazon S3에서 제공하고 있고 보면 8.1이 있는 것을 볼 수 있다.

 ![img3](/assets/img/2024-11-06-rocky-linux-8-10-docker-install/img3.png)

&nbsp;뭐야 있는데 왜 안되지? 싶었는데 다시 생각해보면 8.1이 아니라 **8.10**이다. 뒤에 숫자를 8.10으로 바꾸면 에러메세지에서 보았던 정겨운 404 Not Found를 볼 수 있다.

 ![img4](/assets/img/2024-11-06-rocky-linux-8-10-docker-install/img4.png)

&nbsp;으흠~ 이래서 안됐었다. 8.x를 누르면 301을 받고 8로 리다이렉트가 되는데, 8.xx에 대한 리다이렉트는 없기 때문이다. 그렇다면 나도 8로 쓰면 될 것 같다.
 
### 버전 바꾸기

&nbsp;직접적인 에러 해결 글은 아니지만 [StackOverFlow 글](https://stackoverflow.com/questions/70358656/rhel8-fedora-yum-dnf-causes-cannot-download-repodata-repomd-xml-for-docker-ce)에서 /etc/yum.repos.d/docker-ce.repo를 확인하면 다운로드 링크를 볼 수 있다고 한다. 

 ![img5](/assets/img/2024-11-06-rocky-linux-8-10-docker-install/img5.png)

&nbsp;아까 에러메세지에서 보았던 링크가 있었다. 여기서 $releasever을 8로 바꾸면 되는데.. 한 가지 생각이 들었다. $releasever을 바꿀 수는 없을까? 찾아보니 바로 나왔다. [블로그 글](https://heum-story.tistory.com/142)을 참고했고, 한 줄만에 끝나서 좋았다.

```
$ echo "8" > /etc/yum/vars/releasever
```

&nbsp;그 후 다시 dnf repolist -v를 해보면 예쁘게 나오는 것을 볼 수 있다.

 ![img6](/assets/img/2024-11-06-rocky-linux-8-10-docker-install/img6.png)

 ### 결과

&nbsp;이 후는 블로그를 따라 무난히 설치했다. 사실 원래 서버에는 이미 설치하고 다른 Micro 서버에 테스트한걸 캡쳐한거라 여기서 멈췄다 ㅎ
 
