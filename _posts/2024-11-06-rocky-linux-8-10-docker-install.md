---
title: Rocky Linux 8.10 Docker Install(NCP Micro Server)
categories: [Linux]
tags: [linux, rocky linux, ncp, docker, 리눅스, 록키 리눅스, 도커]
---

&nbsp;기존에 썼던 AWS EC2 Freetier가 만료되어서 새로운 프로젝트는 NCP를 사용해보기로 했다. 계정 몇개 더파서 익숙한 AWS를 쓸까했는데 멘토링을 하는 곳에서 NCP 크레딧을 지급해줘서 그런김에 사용해봤다.

&nbps;프로젝트에서 배치를 돌려서 배치 돌리는 툴로 Jenkins를 사용했는데 전에 Docker Image로 간단히 시작했던 기억이 있어서 이번에도 Docker안에서 Jenkins를 사용하기로 했다. 근데... Docker가 안깔린다...?

### 서버 스펙과 오류
