---
title: "Jenkins 서버 해킹당한 썰"
category: "Jenkins"
tags: ["jenkins", "batch", "hacking"]
published: false
---

### 개요

 Spring Batch를 스케쥴링하는 서버로 쓰고 있는 Jenkins 서버가 있었다. NCP Micro 서버로 무려 vcpu 1개, RAM 1GB를 지니고 있는 서버이다. 이 서버는 사연이 있다... 이미 해킹을 당한 적이 있는 서버였다.

 ![img1](/assets/img/2025-02-24-jenkins-hacking/img1.png)

 root 아이디의 비밀번호를 조금 쉬운걸로 해놨었는데 바로 해킹당했다. AWS에서는 키 파일로 로그인해서 생각도 하지 못하고 있었다가

### 감염 경로
