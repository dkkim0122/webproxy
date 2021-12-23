####################################################################
# CS:APP Proxy Lab
#
# Student Source Files
####################################################################

This directory contains the files you will need for the CS:APP Proxy
Lab.

proxy.c
csapp.h
csapp.c
    These are starter files.  csapp.c and csapp.h are described in
    your textbook. 

    You may make any changes you like to these files.  And you may
    create and handin any additional files you like.

    Please use `port-for-user.pl' or 'free-port.sh' to generate
    unique ports for your proxy or tiny server. 

Makefile
    This is the makefile that builds the proxy program.  Type "make"
    to build your solution, or "make clean" followed by "make" for a
    fresh build. 

    Type "make handin" to create the tarfile that you will be handing
    in. You can modify it any way you like. Your instructor will use your
    Makefile to build your proxy from source.

port-for-user.pl
    Generates a random port for a particular user
    usage: ./port-for-user.pl <userID>

free-port.sh
    Handy script that identifies an unused TCP port that you can use
    for your proxy or tiny. 
    usage: ./free-port.sh

driver.sh
    The autograder for Basic, Concurrency, and Cache.        
    usage: ./driver.sh

nop-server.py
     helper for the autograder.         

tiny
    Tiny Web server from the CS:APP text

-----
<br>

# proxylab


[porxylab.pdf](http://csapp.cs.cmu.edu/3e/proxylab.pdf)

### 프록시 서버란

> 클라이언트와 서버 사이에서 데이터를 전달해주는 서버.
> 

웹 캐시 등에 많이 쓰는데, 클라이언트의 LAN에 같이 위치해 있어 클라이언트가 번거롭게 외부 네트워크까지 가지 않도록 도와준다. 만약 프록시 서버의 데이터에 클라이언트가 요청한 브라우저 데이터가 캐싱되어 있으면 바로 빠르게 클라이언트가 응답을 받을 수 있다.

### 목표

- **write a simple HTTP proxy that caches web objects.**

---

## **Part I: Implementing a sequential web proxy**

- basic sequential proxy that handles HTTP/1.0 GET requests
- read the entirety of the request from the client and parse the request.
- establish its own connection to the appropriate web server then request the object the client specified.
- proxy should read the server’s response and forward it to the client.

### 작동 방법

- 엔드 서버 실행(여기서는 tiny 서버로 하겠음)
    
    ```c
    엔드 서버
    > ./tiny 8000
    ```
    
- 프록시 서버 실행
    
    ```c
    프록시 서버
    > ./proxy 5000
    ```
    
- 클라이언트와 프록시 서버 연결
    
    ```c
    클라이언트
    > telnet localhost 5000
    ```
    
- 클라이언트가 연결하고 싶은 엔드 서버의 정보를 프록시 서버에게 요청
    
    ```c
    클라이언트
    
    GET http://localhost:8000/home.html HTTP/1.1   
    Host : localhost
    Connection: xxx
    User-Agent: yyy
    Proxy-Connection: zzzz
    ```
    
- 엔드 서버 결과
    
    ```c
    엔드 서버
    
    Request headers:
    GET /home.html HTTP/1.0
    Connection: close
    Proxy-Connection: close
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:10.0.3) Gecko/20120305 Firefox/10.0.3
    Connection: xxx
    User-Agent: yyy
    Proxy-Connection: zzzz
    
    Response headers:
    HTTP/1.0 200 OK
    Server: Tiny Web Server
    Connection: close
    Content-length: 250
    Content-type: text/html
    ```
    
- 프록시 서버 결과
    
    ```c
    proxy received 17 bytes,then send
    proxy received 25 bytes,then send
    proxy received 19 bytes,then send
    proxy received 21 bytes,then send
    proxy received 25 bytes,then send
    proxy received 2 bytes,then send
    proxy received 7 bytes,then send
    proxy received 33 bytes,then send
    proxy received 8 bytes,then send
    proxy received 40 bytes,then send
    proxy received 16 bytes,then send
    proxy received 42 bytes,then send
    proxy received 31 bytes,then send
    proxy received 49 bytes,then send
    proxy received 9 bytes,then send
    proxy received 8 bytes,then send
    proxy received 7 bytes,then send
    ```
    
- 클라이언트 결과
    
    ```c
    HTTP/1.0 200 OK
    Server: Tiny Web Server
    Connection: close
    Content-length: 250
    Content-type: text/html
    
    <html>
    <head><title>test</title></head>
    <body> 
    <img align="middle" src="godzilla.gif">
    Dave O'Hallaron
    <video width="320" height="240" controls>
        <source src="snowing.mp4">
        Your browser does not support the video tag.
    </video>
    </body>
    </html>
    ```
    

### 매크로, 변수 및 함수선언

```c
#include <stdio.h>
#include "csapp.h"
/* Recommended max cache and object sizes */
#define MAX_CACHE_SIZE 1049000
#define MAX_OBJECT_SIZE 102400

/* You won't lose style points for including this long line in your code */
static const char *user_agent_hdr = "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:10.0.3) Gecko/20120305 Firefox/10.0.3\r\n";
static const char *conn_hdr = "Connection: close\r\n";
static const char *prox_hdr = "Proxy-Connection: close\r\n";
static const char *host_hdr_format = "Host: %s\r\n";
static const char *requestlint_hdr_format = "GET %s HTTP/1.0\r\n";
static const char *endof_hdr = "\r\n";

static const char *connection_key = "Connection";
static const char *user_agent_key= "User-Agent";
static const char *proxy_connection_key = "Proxy-Connection";
static const char *host_key = "Host";

void doit(int connfd);
void parse_uri(char *uri,char *hostname,char *path,int *port);
void build_http_header(char *http_header,char *hostname,char *path,int port,rio_t *client_rio);
int connect_endServer(char *hostname,int port,char *http_header);
```

### main()

> 프록시 서버에 사용할 포트 번호를 인자로 받아, 프록시 서버가 클라이언트와 연결할 연결 소켓 `connfd`를 만들고 `doit()` 함수 실행.
> 

```c
int main(int argc,char **argv)
{
    int listenfd,connfd;
    socklen_t  clientlen;
    char hostname[MAXLINE],port[MAXLINE];

    struct sockaddr_storage clientaddr;/*generic sockaddr struct which is 28 Bytes.The same use as sockaddr*/

    if(argc != 2){
        fprintf(stderr,"usage :%s <port> \n",argv[0]);
        exit(1);
    }

    listenfd = Open_listenfd(argv[1]);
    while(1){
        clientlen = sizeof(clientaddr);
        connfd = Accept(listenfd,(SA *)&clientaddr,&clientlen);

        /*print accepted message*/
        Getnameinfo((SA*)&clientaddr,clientlen,hostname,MAXLINE,port,MAXLINE,0);
        printf("Accepted connection from (%s %s).\n",hostname,port);

        /*sequential handle the client transaction*/
        doit(connfd);

        Close(connfd);
    }
    return 0;
}
```

### doit()

> 클라이언트의 요청 라인을 파싱해 1) **엔드 서버의 hostname, path, port**를 가져오고, 2) 엔드 서버에 보낼 **요청 라인과 헤더를 만들 변수들을 만듦**. 3) **프록시 서버와 엔드 서버를 연결**하고 엔드 서버의 **응답 메세지를 클라이언트에** 보내줌.
> 

**변수**

- **end_serverfd** : 프록시 서버와 엔드 서버를 이어주는 프록시 서버 측 클라이언트 소켓
- **hostname, path, port :** 프록시 서버가 파싱한 엔드 서버의 정보(클라이언트가 요청함)
- **endserver_http_header** : 엔드 서버에 보낼 프록시 서버의 요청 라인과 헤더

```c
void doit(int connfd)
{
    int end_serverfd;/*the end server file descriptor*/

    char buf[MAXLINE],method[MAXLINE],uri[MAXLINE],version[MAXLINE];
    char endserver_http_header [MAXLINE];
    /*store the request line arguments*/
    char hostname[MAXLINE],path[MAXLINE];
    int port;

    rio_t rio,server_rio;/*rio is client's rio,server_rio is endserver's rio*/

    /* 클라이언트가 보낸 요청 헤더에서 method, uri, version을 가져옴.*/
    /* GET http://localhost:8000/home.html HTTP/1.1 */
    Rio_readinitb(&rio,connfd);
    Rio_readlineb(&rio,buf,MAXLINE);
    sscanf(buf,"%s %s %s",method,uri,version); /*read the client request line*/

    if(strcasecmp(method,"GET")){
        printf("Proxy does not implement the method");
        return;
    }

    /*parse the uri to get hostname,file path ,port*/
    /* 프록시 서버가 엔드 서버로 보낼 정보들을 파싱함. */
    // hostname -> localhost, path -> /home.html, port -> 8000
    parse_uri(uri,hostname,path,&port);

    /*build the http header which will send to the end server*/
    /* 프록시 서버가 엔드 서버로 보낼 요청 헤더들을 만듦. endserver_http_header가 채워진다. */
    build_http_header(endserver_http_header,hostname,path,port,&rio);

    /*connect to the end server*/
    /* 프록시 서버와 엔드 서버를 연결함 */
    end_serverfd = connect_endServer(hostname,port,endserver_http_header);
    // clinetfd connected from proxy to end server at proxy side
    // port: 8000
    if(end_serverfd<0){
        printf("connection failed\n");
        return;
    }

    /* 엔드 서버에 HTTP 요청 헤더를 보냄 */
    Rio_readinitb(&server_rio,end_serverfd);
    /*write the http header to endserver*/
    Rio_writen(end_serverfd,endserver_http_header,strlen(endserver_http_header));

    /* 엔드 서버로부터 응답 메세지를 받아 클라이언트에 보내줌. */
    /*receive message from end server and send to the client*/
    size_t n;
    while((n=Rio_readlineb(&server_rio,buf,MAXLINE))!=0)
    {
        printf("proxy received %d bytes,then send\n",n);
        Rio_writen(connfd,buf,n); // connfd -> client와 proxy 연결 소켓. proxy 관점.
    }
    Close(end_serverfd);
}
```

### build_http_header()

> 클라이언트로부터 받은 요청 헤더를 정제해서 프록시 서버가 엔드 서버에 보낼 요청 헤더를 만듦.
> 
- 각 변수들의 값 → 클라이언트 request가 `“GET [http://localhost:8000/home.html](http://localhost:8000/home.html) HTTP/1.1”` 일 때
    - request_hdr = `"GET /home.html HTTP/1.0\r\n"`
    - host_hdr = `"Host: localhost:8000"`
    - conn_hdr = `"Connection: close\r\n"`
    - prox_hdr = `"Proxy-Connection: close\r\n"`
    - user_agent_hdr = `"User-Agent: ...."`
    - other_hdr = `Connection, Proxy-Connection, User-Agent`가 아닌 모든 헤더

```c
void build_http_header(char *http_header,char *hostname,char *path,int port,rio_t *client_rio)
{
    char buf[MAXLINE],request_hdr[MAXLINE],other_hdr[MAXLINE],host_hdr[MAXLINE];

    /* 응답 라인 만들기 */
    sprintf(request_hdr,requestlint_hdr_format,path);

    /* 클라이언트 요청 헤더들에서 Host header와 나머지 header들을 구분해서 넣어줌 */
    /*get other request header for client rio and change it */
    while(Rio_readlineb(client_rio,buf,MAXLINE)>0)
    {
        if(strcmp(buf,endof_hdr)==0) break; /* EOF, '\r\n' 만나면 끝 */

        /* 호스트 헤더 찾기 */
        if(!strncasecmp(buf,host_key,strlen(host_key)))/*Host:*/ //일치하는 게 있으면 0
        {
            strcpy(host_hdr,buf);
            continue;
        }
        /* 나머지 헤더 찾기 */
        if(strncasecmp(buf,connection_key,strlen(connection_key))
              && strncasecmp(buf,proxy_connection_key,strlen(proxy_connection_key))
              && strncasecmp(buf,user_agent_key,strlen(user_agent_key)))
        {
            strcat(other_hdr,buf);
        }
    }
    if(strlen(host_hdr)==0)
    {
        sprintf(host_hdr,host_hdr_format,hostname);
    }

    /* 프록시 서버가 엔드 서버로 보낼 요청 헤더 작성 */
    sprintf(http_header,"%s%s%s%s%s%s%s",
            request_hdr,
            host_hdr,
            conn_hdr,
            prox_hdr,
            user_agent_hdr,
            other_hdr,
            endof_hdr);

    return ;
}
```

### connect_endServer()

> 프록시 서버와 엔드 서버를 연결한다.
> 

```c
inline int connect_endServer(char *hostname,int port,char *http_header){
    char portStr[100];
    sprintf(portStr,"%d",port);
    return Open_clientfd(hostname,portStr);
}
```

### parst_uri()

> 클라이언트의 uri를 파싱해 서버의 hostname, path, port를 찾는다.
> 
- 변수들 → 클라이언트 request가 `“GET [http://localhost:8000/home.html](http://localhost:8000/home.html) HTTP/1.1”` 일 때
    - hostname = `localhost`
    - path = `/home.html`
    - port = `8000`

```c
void parse_uri(char *uri,char *hostname,char *path,int *port)
{
    *port = 80; // default port
    char* pos = strstr(uri,"//");  /* http://이후의 string들 */

    pos = pos!=NULL? pos+2:uri;  /* http:// 없어도 가능 */ 

    /* port와 path를 파싱 */
    char*pos2 = strstr(pos,":");
    if(pos2!=NULL)
    {
        *pos2 = '\0';
        sscanf(pos,"%s",hostname);
        sscanf(pos2+1,"%d%s",port,path); // port change from 80 to client-specifying port
    }
    else
    {
        pos2 = strstr(pos,"/");
        if(pos2!=NULL)
        {
            *pos2 = '\0';
            sscanf(pos,"%s",hostname);
            *pos2 = '/';
            sscanf(pos2,"%s",path);
        }
        else
        {
            sscanf(pos,"%s",hostname);
        }
    }
    return;
}
```
<br>
<br>

# **Part II: Dealing with multiple concurrent requests**

- The simplest way to implement a concurrent server is to spawn a new thread to handle each new connection request.
- Note that your threads should run in detached mode to avoid memory leaks.

<br>

## posix 쓰레드

> UNIX 기반 OS 시스템에 API로 정의되어있는 Thread이다. `<pthread.h>` 헤더파일을 include해서 쓴다.
> 

리눅스에서는 gcc -o thread thread.c **-lpthread**를 뒤에 꼭 적어주어야 한다.

<br>

## 쓰레드 생성 pthread_create()

> 쓰레드를 생성한다.
> 

```c
#include <pthread.h>
typedef void *(func)(void *);

int pthread_create(pthread_t *tid, pthread_attr_t *attr, func *f, void *arg);
```

**인자**

- **쓰레드의 고유 식별자 tid**
- **쓰레드 특성 attr** : 기본 쓰레드 특성을 이용하려면 **NULL**을 사용한다.
- **쓰레드가 수행하게 될 함수 f**
- **함수 f에 넣게 될 인자 arg**

<br>

## pthread_self()

> 지금 문맥 상에서 실행되고 있는 쓰레드의 식별자 번호를 리턴한다.
> 

```c
pthread_t tid;

tid = pthread_self();
```

<br>

## 쓰레드 종료 pthread_exit()

> **해당 쓰레드를 종료한다.**
> 

```c
void pthread_exit(void *thread_return);
```

**인자**

- **thread_return**

### 종료한 쓰레드 삭제 pthread_join()

> 인자로 넣어 준 **자식 쓰레드가 종료될 때까지 기다린다.** 그 후 **종료된 쓰레드가 가지고 있는 모든 메모리 자원을 삭제한다.**
> 

```c
int pthread_join(pthread_t thread, void **thread_return);
```

**인자**

- **기다리고자 하는 자식 쓰레드의 식별자 thread**
- **자식 쓰레드가 종료 시 리턴되는 값 thread_return**

<br>

## 쓰레드 분리 pthread_detach()

> 쓰레드를 분리시킨다.
> 

```c
int pthread_detach (pthread_t tid);
```

기본적으로 연결된 쓰레드는 종료되더라도 pthread_join()을 사용하기 전에는 자원이 삭제(회수)되지 않는다. 하지만 **분리된 쓰레드를 종료시킬 경우 그 즉시 자원이 회수될 수 있다.**

**인자**

- **분리시킬 쓰레드 tid** : 자기 자신을 분리시킬 경우 `pthread_self()`**의 리턴값을 넣는다.**

<br>

# proxy 과제에 적용하기

## proxy.c

### 변수 선언

```c
/* 쓰레드가 생성될 때 수행하게 될 함수를 선언한다. */
void* thread(void* vargsp);
```

## main()

클라이언트를 연결할 때마다 그 연결을 수행하는 쓰레드를 만들어준다.

```c
int main(int argc,char **argv)
{
    int listenfd, *connfdp; 
    socklen_t  clientlen;
    char hostname[MAXLINE],port[MAXLINE];
    pthread_t tid;  /* 새로 만들어진 쓰레드의 식별자 */

    struct sockaddr_storage clientaddr;/*generic sockaddr struct which is 28 Bytes.The same use as sockaddr*/

    if(argc != 2){
        fprintf(stderr,"usage :%s <port> \n",argv[0]);
        exit(1);
    }

    /* 연결이 끊어진 소켓에 데이터를 전달해주면 SIGPIPE 시그널이 전달되면서 프로세스가 바로 종료된다. */
    /* SIGPIPE 시그널을 무시(SIG_IGN)해준다는 처리를 해 주어야 한다. */
    Signal(SIGPIPE, SIG_IGN);
    listenfd = Open_listenfd(argv[1]);

    /* 연결 시마다 쓰레드를 만들어준다. */
    while(1){
        clientlen = sizeof(clientaddr);

        /* pthread_create의 경우 argp 인자가 void* 이다. 
        따라서 연결 식별자를 인자로 넣어줄 수 있게 안전하게 포인터를 만들어준다. */
        connfdp = Malloc(sizeof(int));
        *connfdp = Accept(listenfd,(SA *)&clientaddr,&clientlen); // 포인터가 가리키는 값을 연결 식별자 값으로.

        /*print accepted message*/
        Getnameinfo((SA*)&clientaddr,clientlen,hostname,MAXLINE,port,MAXLINE,0);
        printf("Accepted connection from (%s %s).\n",hostname,port);

        /*sequential handle the client transaction*/
        //doit(connfd);
        //Close(connfd);

        /* doit과 Close를 쓰레드 안에서 수행할 것이다. */
        /* thread 함수를 connfd 인자로 받아서. */
        Pthread_create(&tid, NULL, thread, connfdp);
    }
    return 0;
}
```

## thread()

> 새롭게 생성된 쓰레드 안에서 클라이언트와의 통신을 수행한다.
> 

인자로 연결 식별자를 받는다. 자기 자신을 먼저 분리한 다음 doit()을 수행한다. 그 후 바로 종료한다. 분리되어 있으므로 종료 후 즉시 메모리가 회수된다.

```c
void* thread(void *vargp){
    int connfd = *((int*)vargp);
    Pthread_detach(pthread_self());  // 자기 자신을 분리해준다.
    Free(vargp);  // 동적 할당한 파일 식별자 포인터를 free해준다.
    doit(connfd);
    Close(connfd);
}
```