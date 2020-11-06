---
layout: post
title: "[pwnable.kr] flag 풀이"
date: 2020-11-06
categories: PWN
tags: [pwn,wargame,system]
---

## 문제풀이
Exeinfo PE 를 통해 패킹되어있는지 확인 후 upx를 이용하여 언패킹한다.

## UPX 명령어
upx -d [파일명] -o [언패킹된 파일 이름]
[upx 다운로드](https://github.com/upx/upx/releases/tag/v3.96)
최신 upx 가 아닐 경우 에러가 발생하니 3.9x 이상을 다운로드해야함.

## IDA main 부분
```
text:0000000000401164 ; =============== S U B R O U T I N E =======================================
.text:0000000000401164
.text:0000000000401164 ; Attributes: bp-based frame
.text:0000000000401164
.text:0000000000401164 ; int __cdecl main(int argc, const char **argv, const char **envp)
.text:0000000000401164                 public main
.text:0000000000401164 main            proc near               ; DATA XREF: _start+1D↑o
.text:0000000000401164
.text:0000000000401164 dest            = qword ptr -8
.text:0000000000401164
.text:0000000000401164                 push    rbp
.text:0000000000401165                 mov     rbp, rsp
.text:0000000000401168                 sub     rsp, 10h
.text:000000000040116C                 mov     edi, offset aIWillMallocAnd ; "I will malloc() and strcpy the flag there "... # flag 변수에 flag값이 있음을 암시
.text:0000000000401171                 call    puts
.text:0000000000401176                 mov     edi, 64h
.text:000000000040117B                 call    malloc
.text:0000000000401180                 mov     [rbp+dest], rax
.text:0000000000401184                 mov     rdx, cs:flag
# flag값을 더블클릭하여 따라가본다
.text:000000000040118B                 mov     rax, [rbp+dest]
.text:000000000040118F                 mov     rsi, rdx        ; src
.text:0000000000401192                 mov     rdi, rax        ; dest
.text:0000000000401195                 call    _strcpy
.text:000000000040119A                 mov     eax, 0
.text:000000000040119F                 leave
.text:00000000004011A0                 retn
.text:00000000004011A0 main            endp
.text:00000000004011A0
.text:00000000004011A0 ; ---------------------------------------------------------------------------
```

```
data:00000000006C2070 flag            dq offset aUpxSoundsLikeA
.data:00000000006C2070                                         ; DATA XREF: main+20↑r
.data:00000000006C2070                                         ; 'UPX...? sounds like a delivery service :)'... 

# flag값 발견!
```


!! 리버싱할 때는 항상 패킹되어있는지 확인하기 !!