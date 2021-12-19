**06. echo 클라이언트와 서버**
===

<br>

**RIO(Robust I/O)**
===

[참고 : 11. System-Level I/O](https://suhwanc.tistory.com/131)

**RIO 버퍼 없는 입력 및 출력 함수**
----

> 메모리와 파일 간에 직접 데이터를 전송할 수 있다.
> 
- **rio_readn**
    
    > 현재 파일 식별자 fd에서 n 만큼의 데이터를 버퍼 usrbuf로 보낸다.
    > 
    
    ```c
    #include "csapp.h"
    
    ssize_t rio_readn(int fd, void* usrbuf, size_t n);
    ```
    
    - **파라미터**
        - **내가 보낼 바이트가 저장된 현재 파일 위치 fd**
        - **파일에서 전송할 대상 메모리 버퍼 위치 usrbuf**
        - **전송할 바이트 수 n**
    - **리턴값**
        - 성공하면 전송한 바이트 수, EOF를 읽기 중에 만나면 0, 에러가 나면 -1

<br>

- **rio_writen**
    
    > 현재 메모리의 버퍼 usrbuf에서 n 만큼의 데이터를 파일 식별자 fd로 보낸다.
    > 
    
    ```c
    #include "csapp.h"
    
    ssize_t rio_writen(int fd, void* usrbuf, size_t n);
    ```
    
    - **파라미터**
        - **내가 n 바이트를 쓰고 싶은 현재 파일 위치 fd**
        - **n 바이트가 있는 메모리 버퍼 위치 usrbuf**
        - **전송할 바이트 수 n**
    - **리턴값**
        - 성공하면 전송한 바이트 수, 에러가 나면 -1

<br>

**RIO 버퍼를 통한 입력 함수**
---

> 텍스트 라인 전체를 내부 읽기 버퍼에서 복사하는 `rio_readlineb`와 텍스트 라인과 바이너리 데이터 모두를 읽을 수 있는 `rio_readnb`.
> 

파일 → 읽기 버퍼 → (읽기 버퍼를 읽고 그 데이터를)메모리 버퍼로 복사

- **rio_readinitb**
    
    > 읽고 싶은 파일 식별자 `fd`와 읽기 버퍼 `rp`를 연결한다.
    > 
    
    ```c
    #include "csapp.h"
    
    void rio_readinitb(rio_t* rp, int fd);
    ```
    
- **rio_readlineb**
    
    > 텍스트 라인 전체를 내부 읽기 버퍼 `rp`에서 읽은 후, 메모리 버퍼 `usrbuf`으로 복사하고, `\0(NULL)`로 텍스트 라인을 종료시킨다.
    > 
    
    최대 `maxlen-1` 바이트 만큼 읽고 마지막 1바이트는 `\0`을 넣어준다.
    
    ```c
    ssize_t rio_readlineb(rio_t* rp, void* usrbuf, size_t maxlen);
    ```
    

- **rio_readnb**
    
    > 텍스트 라인과 바이너리 데이터 모두를 읽을 수 있다. n 바이트씩 가져온다.
    > 
    
    ```c
    ssize_t rio_readnb(rio_t* rp, void* usrbuf, size_t n);
    ```
    
<br>

**echoclient.c**
===

> 서버와의 연결 후 클라이언트와 통신한다.
> 
- **0번째 인자로 실행 파일, 1번째로 호스트네임, 2번째로 포트 넘버를 받는다.**
    
    ```c
    #include "csapp.h"
    
    /* 0번째 인자로 실행 파일, 1번째로 호스트네임, 2번째로 포트 넘버를 받는다.*/
    int main(int argc, char **argv)
    {
        int clientfd;
        char *host, *port, buf[MAXLINE];
        rio_t rio;
    
        host = argv[1];
        port = argv[2];
    ```
    
- **서버와의 연결을 성공한다(connect까지)**
    
    ```c
        clientfd = Open_clientfd(host, port);  // 서버와의 연결 성공(connect까지
    ```
    
- **클라이언트 소켓 파일 식별자와 읽기 버퍼 rio를 연결한다.**
    
    ```c
        /* 1. 클라이언트 소켓 파일 식별자와 읽기 버퍼 rio를 연결한다.*/
        Rio_readinitb(&rio, clientfd);
    ```
    
- **표준 입력을 통해 읽기 버퍼 rio에 데이터를 쓴다. 또 서버가 쓴 rio의 데이터를 읽어 표준 출력한다.**
    
    ```c
        /* 표준 입력에서 텍스트 줄을 반복적으로 읽는다. */
        /* 2. 표준 입력sdtin에서 MAXLINE만큼 바이트를 가져와 buf에 저장한다. */
        while (Fgets(buf, MAXLINE, stdin) != NULL) {  // 6. EOF 표준 입력을 만나면 종료한다.
            // 3. buf 메모리 안의 strlen(buf) 바이트 만큼의(사실상 모두)를 clientfd로 보낸다.
            Rio_writen(clientfd, buf, strlen(buf));  
            // 4. 서버가 rio에 echo줄을 쓰면 그 rio를 읽어서 읽기 버퍼 buf에 쓴다.
            Rio_readlineb(&rio, buf, MAXLINE);
            // 5. buf에 받아온 값을 표준 출력으로 인쇄한다.
            Fputs(buf, stdout);   
        }
    ```
    
- **표준 입력에 EOF를 입력해 루프를 종료하고 클라이언트 식별자를 닫는다. 그 후 클라이언트를 종료한다.**
    
    <aside>
    💡 클라이언트 식별자를 닫으면 서버에 EOF 통지가 간다. 그럼 서버에서 rio_readlineb의 리턴값이 0이 된다.
    
    </aside>
    
    ```c
        Close(clientfd); // 루프가 종료되면 클라이언트 식별자를 닫는다. 서버에 EOF 통지가 전송된다.
        exit(0); // 클라이언트가 종료된다.
    }
    ```

<br>

**echoserver.c**
===

> 서버가 클라이언트의 연결 요청을 받아 연결 소켓 식별자를 만든다. 그리고 echo함수를 호출한다.
> 
- **서버의 포트 번호를 1번째 인자로 받는다.**
    
    > 클라이언트 연결 소켓 주소 구조체를 담을 때 크기가 충분히 큰sockaddr_storage 구조체를 사용해 프로토콜 독립적으로 프로그램을 만든다.
    > 
    
    ```c
    #include "csapp.h"
    
    void echo(int connfd);
    
    /* 서버의 포트 번호를 1번째 인자로 받는다. */
    int main(int argc, char **argv)
    {
        int listenfd, connfd;
        socklen_t clientlen;
        /* Accept로 보내지는 client 소켓 구조체. */
        struct sockaddr_storage clientaddr; /* sockaddr_storage 구조체: 모든 프로토콜의 주소에 대해 Enough room for any addr */                                                                                                               
        char client_hostname[MAXLINE], client_port[MAXLINE];
    
        // 인자 2개 다 받아야 함.
        if (argc != 2){
            fprintf(stderr, "usage: %s <port> \n", argv[0]);
            exit(0);
        }
    
        /* 해당 포트 번호에 적합한 듣기 식별자를 만들어 준다. */
        listenfd = Open_listenfd(argv[1]);
    
    ```
    
- **클라이언트의 연결 요청을 계속 받아서 연결 식별자를 만들고 echo 함수를 호출한다.**
    
    클라이언트 식별자가 닫히면 서버 연결 식별자도 닫아준다.
    
    ```c
        while (1) {
        /* 클라이언트의 연결 요청을 계속 받아서 연결 식별자를 만든다. */
    	clientlen = sizeof(struct sockaddr_storage); /* Important! 길이가 충분히 커서 프로토콜-독립적!*/
        connfd = Accept(listenfd, (SA *)&clientaddr, &clientlen); // 클라이언트와 통신하는 연결 식별자
    	
        /* 클라이언트와 제대로 연결됐다는 것을 출력해준다. */
        Getnameinfo((SA *) &clientaddr, clientlen, 
                        client_hostname, MAXLINE, client_port, MAXLINE, 0);
    	printf("Connected to (%s, %s)\n", client_hostname, client_port);
    	
        /*  */
        echo(connfd);
    
        /* 연결 식별자를 닫아준다. */
    	Close(connfd);
        }
    ```
    
- **서버를 닫아준다.**
    
    ```c
        /* 서버 종료 */
        exit(0);
    }
    ```
    
<br>

**echo.c**
===

> 클라이언트와 연결된 읽기 버퍼에 서버 연결 소켓도 연결해준다. 클라이언트가 보낸 데이터를 그대로 읽기 버퍼에 다시 써 준다.
> 

```c
#include "csapp.h"

void echo(int connfd)
{
    size_t n;
    char buf[MAXLINE];
    rio_t rio;

    /* 읽기 버퍼 rio와 서버의 연결 소켓 식별자를 연결해준다. clientfd도 연결되어 있다. */
    Rio_readinitb(&rio, connfd);

    /* 읽기 버퍼 rio에서 클라이언트가 보낸 데이터를 읽고, rio에 그 데이터를 고대로 쓴다.*/
    /* 읽기 버퍼 rio에서 MAXLINE만큼의 데이터를 읽어 와 buf에 넣는다. */
    while((n = Rio_readlineb(&rio, buf, MAXLINE)) != 0) { // 0이면 클라이언트 식별자가 닫혔다는 소리다.
        printf("server received %d bytes\n", (int)n);

        /* buf 안에는 클라이언트가 보낸 데이터 그대로 있다. */
        /* buf 메모리 안의 클라이언트가 보낸 바이트 만큼의(사실상 모두)를 clientfd로 보낸다. */
    	Rio_writen(connfd, buf, n);
    }

    /* 클라이언트 식별자가 닫히면 루프 종료 및 함수도 종료. */
}
```
<br>

**Makefile**
--

```makefile
CC = gcc
CFLAGS = -O2 -Wall -I .

# This flag includes the Pthreads library on a Linux box.
# Others systems will probably require something different.
LIB = -lpthread

all: echoclient echoserver

echoclient: echoclient.c csapp.o
	$(CC) $(CFLAGS) -o echoclient echoclient.c csapp.o $(LIB)

echoserver: echoserver.c csapp.o echo.o
	$(CC) $(CFLAGS) -o echoserver echoserver.c csapp.o echo.o $(LIB)

csapp.o: csapp.c
	$(CC) $(CFLAGS) -c csapp.c

echo.o: echo.c
	$(CC) $(CFLAGS) -c echo.c
```
<br>

**실행**
---

1. 서버를 열어준다. 서비스에 필요한 포트 번호를 같이 입력한다.
    
    ```c
    $ ./echoserver 8000
    ```
    
2. 클라이언트를 열어준다. 내가 요청을 보내줄 서버의 IP와 포트 번호를 입력한다.

    난 AWS 서버를 사용 중이므로 해당 인스턴스의 Public IP가 서버의 IP이다. 만약 WSL을 사용 중이라면 그냥 내 컴퓨터 호스트의 IP를 사용한다. 아니면 Localhost를 사용해도 된다!
    
    ```c
    $ ./echoclient xxx.xxx.xxx.xxx 8000
    ```
    
3. 연결이 성공적이었다면 서버에 메세지가 뜬다.
    
    ```c
    Connected to (xxx.xxx.xxx.xxx, 41xxx)
    ```
    
4. 클라이언트에서 요청을 보낸다. 이 경우엔 간단한 바이트 입력.
    
    ```c
    hello
    ```
    
5. 서버는 입력받은 데이터의 바이트 수를 출력한다.
    
    ```c
    server received 6 bytes
    ```
    
6. 클라이언트에 입력한 데이터 그대로 출력된다.
    
    ```c
    hello
    hello
    ```