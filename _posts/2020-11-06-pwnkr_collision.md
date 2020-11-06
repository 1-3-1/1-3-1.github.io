---
layout: post
title: "[pwnable.kr] collision 풀이"
date: 2020-11-06
categories: PWN
tags: [pwn,wargame,system]
---

## 문제풀이
```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
        int* ip = (int*)p;
        int i;
        int res=0;
        for(i=0; i<5; i++){
                res += ip[i];
        }
        // 배열에 있는 값들 5개를 하나씩 꺼내서 더함

        return res;
        //더한 값을 리턴
}

int main(int argc, char* argv[]){
        if(argc<2){
                printf("usage : %s [passcode]\n", argv[0]);
                return 0;
        }

        if(strlen(argv[1]) != 20){
                printf("passcode length should be 20 bytes\n");
                return 0;
        }

        if(hashcode == check_password( argv[1] )){
                system("/bin/cat flag");
                return 0;
        }
        else
                printf("wrong passcode.\n");
        return 0;
        }
        // hashcode (0x21DD09EC) 값과 check_password(입력값)의 리턴값이 같을 때 flag 출력.
        // check_password 함수가 입력값 5개를 더해주는 함수 이므로 5개 값의 합이 0x21DD09EC와 같을 때 flag 출력.
```

`0x21DD09EC / 5 = 6c5cec8 * 4 + 0x6C5CECC`


오른쪽 값들을 인자로 넣어주면 된다.
리틀엔디언으로 넣어주어야 하므로 거꾸로 넣어줌.

__결과 !__
```c
col@pwnable:~$ ./col `python -c 'print "\xc8\xce\xc5\x06"*4+"\xcc\xce\xc5\x06"'`

daddy! I just managed to create a hash collision :)
```