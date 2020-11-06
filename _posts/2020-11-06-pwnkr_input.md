---
layout: post
title: "[pwnable.kr] input 풀이"
date: 2020-11-06
categories: PWN
tags: [pwn,wargame,system]
---


## 문제풀이
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
        printf("Welcome to pwnable.kr\n");
        printf("Let's see if you know how to give input to program\n");
        printf("Just give me correct inputs then you will get the flag :)\n");

        // argv
        if(argc != 100) return 0;
        if(strcmp(argv['A'],"\x00")) return 0;
        if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
        printf("Stage 1 clear!\n");

        // stdio
        char buf[4];
        read(0, buf, 4);
        if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
        read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
        printf("Stage 2 clear!\n");

        // env
        if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
        printf("Stage 3 clear!\n");

        // file
        FILE* fp = fopen("\x0a", "r");
        if(!fp) return 0;
        if( fread(buf, 4, 1, fp)!=1 ) return 0;
        if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
        fclose(fp);
        printf("Stage 4 clear!\n");

        // network
        int sd, cd;
        struct sockaddr_in saddr, caddr;
        sd = socket(AF_INET, SOCK_STREAM, 0);
        if(sd == -1){
                printf("socket error, tell admin\n");
                return 0;
        }
        saddr.sin_family = AF_INET;
        saddr.sin_addr.s_addr = INADDR_ANY;
        saddr.sin_port = htons( atoi(argv['C']) );
        if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
                printf("bind error, use another port\n");
                return 1;
        }
        listen(sd, 1);
        int c = sizeof(struct sockaddr_in);
        cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
        if(cd < 0){
                printf("accept error, tell admin\n");
                return 0;
        }
        if( recv(cd, buf, 4, 0) != 4 ) return 0;
        if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
        printf("Stage 5 clear!\n");

        // here's your flag
        system("/bin/cat flag");
        return 0;
}
```



### STAGE 1 :
```
// argv
if(argc != 100) return 0; // 인자가 100개인지 확인
if(strcmp(argv['A'],"\x00")) return 0; 
// argv['A']에 \x00 대입
if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
//argv['B']에 \x20\x0a\x0d 대입
printf("Stage 1 clear!\n");
```

strcmp 함수는 두 문자열이 같으면 거짓을 반환하므로 두 문자열을 같게 만들어주어야 한다.

argv[‘A’] 에서 ‘A’는 10진수로 했을대 65 이므로 argv[65]에 \x00 값을 넣어줘도 된다.


페이로드는 다음과 같다.
```python
from pwn import *

#pay = ssh(user="input2",host="pwnable.kr",port=2222,password="guest")
# 이렇게 로컬에서 연결해서 해도됨! process 명령어 쓸 때 pay.process(어쩌구) 이렇게 하면됨.

# STAGE 1

arg = [str(i) for i in range(100)]
arg[65] = "\x00" # arg[ord('A')] = "\x00"
arg[66] = "\x20\x0a\x0d" # arg[ord('B')]  = "\x20\x0a\x0d"

p = process(executable='/home/input2/input',argv=arg)
p.interactive()
```



### STAGE 2 :
```c
// stdio
char buf[4];
read(0, buf, 4);
if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
// 표준 입력의 4바이트를 읽어와서 "\x00\x0a\x00\xff"와 비교

read(2, buf, 4);
if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
// 표준 에러의 4바이트를 읽어와서 "\x00\x0a\x02\xff"와 비교

printf("Stage 2 clear!\n");
```

read 로 입력을 받은 후 값을 비교하는 코드이다.

첫 번째는 fd가 0, 즉 표준 입력을 의미한다. 따라서 사용자의 입력값을 \x00\x0a\x00\xff 로 주면 된다.

두번째는 fd가 2 이므로 표준 에러를 의미한다. 즉 에러메시지가 \x00\x0a\x02\xff 인지 확인하는 것이다. 표준입력보다 복잡하지만, 에러 메시지를 파일에 작성한 뒤 , 표준에러를 해당 파일로 지정해주면 된다.


페이로드는 다음과 같다

```python
from pwn import *

#p = ssh(user="input2",host="pwnable.kr",port=2222,password="guest")
# 이렇게 로컬에서 연결해서 해도됨! process 명령어 쓸 때 p.process(어쩌구) 이렇게 하면됨.

# STAGE 1

arg = [str(i) for i in range(100)]
arg[65] = "\x00" # arg[ord('A')] = "\x00"
arg[66] = "\x20\x0a\x0d" # arg[ord('B')]  = "\x20\x0a\x0d"



# STAGE 2

arg2 = "\x00\x0a\x00\xff"
with open('/tmp/20200909/mystderr','w') as f:
        f.write("\x00\x0a\x02\xff")

p = process(executable='/home/input2/input',argv=arg, stderr=open('/tmp/20200909/mystderr'))

p.sendline(arg2)
p.interactive()
```

### STAGE 3 :
```c
// env
if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
printf("Stage 3 clear!\n");
```
환경변수 \xde\xad\xbe\xef 값을 가져와 “\xca\xfe\xba\xbe”로 설정되어있는지 확인한다.

페이로드는 다음과 같다.
```python
from pwn import *

#p = ssh(user="input2",host="pwnable.kr",port=2222,password="guest")
# 이렇게 로컬에서 연결해서 해도됨! process 명령어 쓸 때 p.process(어쩌구) 이렇게 하면됨.

# STAGE 1

arg = [str(i) for i in range(100)]
arg[65] = "\x00" # arg[ord('A')] = "\x00"
arg[66] = "\x20\x0a\x0d" # arg[ord('B')]  = "\x20\x0a\x0d"



# STAGE 2

arg2 = "\x00\x0a\x00\xff"
with open('/tmp/20200909/mystderr','w') as f:
        f.write("\x00\x0a\x02\xff")


# STAGE 3

arg3 = {"\xde\xad\xbe\xef":"\xca\xfe\xba\xbe"}
# 환경변수는 딕셔너리 타입으로 해야함.

p = process(executable='/home/input2/input',argv=arg, stderr=open('/tmp/20200909/mystderr'),env=arg3)



p.sendline(arg2)
p.interactive()
```


### STAGE 4 :
```c
// file
FILE* fp = fopen("\x0a", "r");
if(!fp) return 0;
if( fread(buf, 4, 1, fp)!=1 ) return 0;
if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
fclose(fp);
printf("Stage 4 clear!\n");
```

파일 “\x0a” 에서 4바이트만큼 읽어와 buf에 저장한 후 그 값이 \x00\x00\x00\x00와 같은 값인지 비교하는 코드이다.
간단하게 \x0a 파일에 \x00\x00\x00\x00 값을 저장하면 된다

페이로드는 다음과 같다
```python
from pwn import *

#p = ssh(user="input2",host="pwnable.kr",port=2222,password="guest")
# 이렇게 로컬에서 연결해서 해도됨! process 명령어 쓸 때 p.process(어쩌구) 이렇게 하면됨.

# STAGE 1

arg = [str(i) for i in range(100)]
arg[65] = "\x00" # arg[ord('A')] = "\x00"
arg[66] = "\x20\x0a\x0d" # arg[ord('B')]  = "\x20\x0a\x0d"



# STAGE 2

arg2 = "\x00\x0a\x00\xff"
with open('/tmp/20200909/mystderr','w') as f:
        f.write("\x00\x0a\x02\xff")


# STAGE 3

arg3 = {"\xde\xad\xbe\xef":"\xca\xfe\xba\xbe"}
# 환경변수는 딕셔너리 타입으로 해야함.


# STAGE 4 
with open('/tmp/20200909/\x0a','w') as f:
        f.write("\x00\x00\x00\x00")


p = process(executable='/home/input2/input',argv=arg, stderr=open('/tmp/20200909/mystderr'),env=arg3)


p.sendline(arg2)
p.interactive()
```


### STAGE 5
```c
// network
int sd, cd;

//소켓 주소의 틀을 형성해주는 구조체.
struct sockaddr_in saddr, caddr;

//TCP 연결 지향형이고 ipv4 도메인을 위한 소켓생성
sd = socket(AF_INET, SOCK_STREAM, 0);
if(sd == -1){
       printf("socket error, tell admin\n");
        return 0;
}

// ip주소와 포트지정
saddr.sin_family = AF_INET;  // 타입 : ipv4
saddr.sin_addr.s_addr = INADDR_ANY;  // IP 주소
saddr.sin_port = htons( atoi(argv['C']) ); // PORT 지정

// 소켓과 서버 주소 바인딩하고 확인
if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
        printf("bind error, use another port\n");
        return 1;
}

// 연결 대기열 1개 생성
listen(sd, 1);

// 요청이 오면 연결 수락.
int c = sizeof(struct sockaddr_in);
cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
if(cd < 0){
        printf("accept error, tell admin\n");
        return 0;
}


if( recv(cd, buf, 4, 0) != 4 ) return 0;
if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
printf("Stage 5 clear!\n");
```

‘C’ 번째 인자 즉, 67번 인자를 포트번호로 삼고, 해당 포트를 열어 받은 값을 buf 에 저장한다.

따라서 c번째 인자에 포트번호를 임의로 지정해주고, 해당포트를 열어 “\xde\xad\xbe\xef” 값을 전송해주면 된다


페이로드는 다음과 같다
```python
from pwn import *

#p = ssh(user="input2",host="pwnable.kr",port=2222,password="guest")
# 이렇게 로컬에서 연결해서 해도됨! process 명령어 쓸 때 p.process(어쩌구) 이렇게 하면됨.

# STAGE 1

arg = [str(i) for i in range(100)]
arg[65] = "\x00" # arg[ord('A')] = "\x00"
arg[66] = "\x20\x0a\x0d" # arg[ord('B')]  = "\x20\x0a\x0d"



# STAGE 2

arg2 = "\x00\x0a\x00\xff"
with open('/tmp/20200909/mystderr','w') as f:
        f.write("\x00\x0a\x02\xff")


# STAGE 3

arg3 = {"\xde\xad\xbe\xef":"\xca\xfe\xba\xbe"}
# 환경변수는 딕셔너리 타입으로 해야함.


# STAGE 4 
with open('/tmp/20200909/\x0a','w') as f:
        f.write("\x00\x00\x00\x00")


# STAGE 5

# arg[67] = '12345'
# 왜인지 모르겠지만 위처럼 하면안됨.
arg[org('C')] = '12345'



p = process(executable='/home/input2/input',argv=arg, stderr=open('/tmp/20200909/mystderr'),env=arg3)


conn = remote('localhost','12345')
conn.send("\xde\xad\xbe\xef")

p.sendline(arg2)
p.interactive()

```


5가지 스테이지를 클리어하면 다음과 같은 결과가 출력된다.
```
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
[*] Got EOF while reading in interactive
```

아마 다음의 코드에서 오류가 나서 안나오는 것 같다

```
// here's your flag
system("/bin/cat flag");
```
현재 폴더에 flag 라는 파일이 없어서 오류가 났기 때문에 심볼릭링크를 걸어주면 정상적으로 출력된다.
```
ln -s /home/input2/flag flag
```


__결과!__
```
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
Mommy! I learned how to pass various input in Linux :)
```



## 함수 정리
다음에 나오는 함수는 꼭 기억해둘것!

# 1 main 함수
`int main(int argc, char* argv[], char* envp[]){`

int argc : 변수의 갯수를 의미
char* argv[] : 프로그램으로 전달된 실제 인수의 값.
char* envp[] : 운영체제의 환경변수를 넘겨받는다.

# 2 read 함수
`ssize_t read (int fd, void *buf, size_t nbytes)`

int fd : 열린 파일을 가리키는 파일 지정번호
void *buf : 파일에서 읽은 데이터를 저장할 메모리공간
size_t nbytes : 읽을 데이터 크기
read 함수의 첫번째 인자인 fd는 다음과 같은 의미를 갖고있다.

0: 표준입력
1: 표준출력
2: 표준에러

# 3 memcmp 함수
`int memcmp(const void *base1, const void *base2, size_t n)`

const void *base1 : 비교대상 데이터가 있는 버퍼
const void *base2 : 비교대상 데이터가 있는 버퍼
size_t n : 비교할 바이트 수
반환값 : 같을때 0 base1 이 크면 양수 base2 가 크면 음수.

# 4 with open as f:
파일 입출력 시 open 과 close를 해주어야 하는데 이렇게 쓰면 close를 자동으로 해줌.

# 5 fread 함수
`size_t fread(void* ptr, size_t size, size_t count, FILE* stream);`

```
FILE* fp = fopen("test.txt", "r");
int a = fread(buf, 4, 1, fp)
```
다음 예시를 통해 설명하자면, test.txt 에서 읽어온 파일스트림(fp)을 4바이트씩 1번 불러와서 buf에 저장하는 코드이다.
즉, 파일스트림에서 size만큼의 byte 를 count 만큼 읽어와 ptr 에 저장하는 함수이다.




## 심볼릭 링크
`ln -s [원본파일][새로만들파일이름]`

링크를 연결하는 것. 쉽게 말하면 바로가기 같은거다.
