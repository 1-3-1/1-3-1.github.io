---
layout: post
title: "[pwnable.kr] random 풀이"
date: 2020-11-06
categories: PWN
tags: [pwn,wargame,system]
---

## 문제풀이
```c
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();        // random value!

        unsigned int key=0;
        scanf("%d", &key);

        if( (key ^ random) == 0xdeadbeef ){
                printf("Good!\n");
                system("/bin/cat flag");
                return 0;
        }

        printf("Wrong, maybe you should try 2^32 cases.\n");
        return 0;
}
```
사용자가 입력한 key 값이랑 랜덤함수인 rand() 함수 값이랑 XOR 하여 0xdeadbeef 값이 나오면 키값을 주는 문제이다.

rand() 함수는 항상 같은 값이 나온다는 취약점이 있기때문에 random 변수 값을 확인해서 0xdeadbeef와 XOR 하여 key 값을 알아내면 된다.

```
[----------------------------------registers-----------------------------------]
RAX: 0x6b8b4567 // 이 값이 rand() 함수 값.
RBX: 0x0
RCX: 0x7f6505b850a4 --> 0x16a5bce3991539b1
RDX: 0x7f6505b850a8 --> 0x6774a4cd16a5bce3
RSI: 0x7fff1644a21c --> 0x6b8b4567
RDI: 0x7f6505b85620 --> 0x7f6505b850b4 --> 0x61048c054e508aaa
RBP: 0x7fff1644a250 --> 0x400670 (<__libc_csu_init>:    mov    QWORD PTR [rsp-0x28],rbp)
RSP: 0x7fff1644a240 --> 0x7fff1644a330 --> 0x1
RIP: 0x400606 (<main+18>:       mov    DWORD PTR [rbp-0x4],eax)
R8 : 0x7f6505b850a4 --> 0x16a5bce3991539b1
R9 : 0x7f6505b85120 --> 0x8
R10: 0x47f
R11: 0x7f65057fbf60 (<rand>:    sub    rsp,0x8)
R12: 0x400510 (<_start>:        xor    ebp,ebp)
R13: 0x7fff1644a330 --> 0x1
R14: 0x0
R15: 0x0
EFLAGS: 0x202 (carry parity adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x4005f8 <main+4>:   sub    rsp,0x10
   0x4005fc <main+8>:   mov    eax,0x0
   0x400601 <main+13>:  call   0x400500 <rand@plt>
=> 0x400606 <main+18>:  mov    DWORD PTR [rbp-0x4],eax
   0x400609 <main+21>:  mov    DWORD PTR [rbp-0x8],0x0
   0x400610 <main+28>:  mov    eax,0x400760
   0x400615 <main+33>:  lea    rdx,[rbp-0x8]
   0x400619 <main+37>:  mov    rsi,rdx
```

main 함수 디버깅을 통해 rand() 함수 값이 0x6b8b4567 임을 확인했다.

`0x6b8b4567 ^ 0xdeadbeef = 0xb526fb88`

scanf 에서 정수형(%d)으로 받고 있으므로 10진수로 바꾸어 입력해주면 된다. 10진수는 3039230856.

__결과!__
```
random@pwnable:~$ ./random

3039230856 //입력값
Good!
Mommy, I thought libc random is unpredictable...
```
