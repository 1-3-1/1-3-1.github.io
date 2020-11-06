---
layout: post
title: "Ping 커맨드 인젝션 공격 "
date: 2020-11-06
categories: WEB_HACKING
tags: [web,웹 취약점,ping,command injection]
---


## 커맨드인젝션이란?
: 웹 어플리케이션에서 시스템 명령을 사용할 때 파이프같은 문구를 이용하여 개발자가 유도한 시스템 명령 이외의 시스템명령어를 실행하는공격.

## ping 커맨드 인젝션
`ping -c 3 입력값`
다음과 같이 ping 을 테스트하는 기능이 있는 웹페이지에서 `"; cat /etc/passwd` 혹은 `"|cat /etc/passwd` 혹은 `"&cat /etc/passwd` 와 같이 입력하여 시스템 내 중요파일을 볼 수 있다.