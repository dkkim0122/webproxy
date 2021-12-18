# **05. 소켓 인터페이스를 위한 도움 함수들**

> 도움 함수들을 이용해 getaddrinfo()와 소켓 인터페이스를 더욱 편리하게 사용할 수 있다.
> 

<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146638076-efec2b67-325a-4df8-8479-621a8bcc2fe3.png" width = "500">
</p>
<p align = 'center'>
<em>open_clientfd과 open_listenfd</em>
</p>

<br>

**open_clientfd**
===

> 클라이언트가 서버와 연결을 설정한다. 즉, **클라이언트의 getaddrinfo부터 connect까지 담당한다.**
> 

💡 **서버의** 호스트 이름(혹은 IP 주소)와 원하는 서비스(포트 번호)를 입력해, **해당 서버의 소켓 주소 구조체의 형식과 맞는 클라이언트의 소켓을 만들고 연결**한다.


```c
#include "csapp.h"

int open_clientfd(char *hostname, char *port);
```

**파라미터**

- **내가 연결하고 싶은 서버의 호스트의 IP주소 hostname**
- **해당 서버 프로세스의 서비스 종류, 즉 포트 번호 port**

**사용되는 함수**

> **addrinfo** 리스트 안의 구조체 멤버들은 모두 **바로바로 소켓 인터페이스 함수들의 인자로 사용**될 수 있다.
> 
- **socket ( 네트워크 주소 체계, 소켓 타입, 프로토콜 종류 )**
- **connect ( 클라이언트의 소켓 번호, 서버의 소켓 주소, 주소의 길이 )**

**리턴값**

- **입력과 출력 준비가 된 열린 클라이언트의 소켓 식별자**

<Br>


**코드**

- **선언**
    
    > 연결을 원하는 **서버** **호스트의 IP 주소(이름)**와 원하는 서비스의 **포트 번호**를 입력받는다.
    > 
    
    ```c
    int open_clientfd(char *hostname, char *port) {
      int clientfd;
      struct addrinfo hints, *listp, *p;
    ```
    
- **getaddrinfo**
    
    > **해당 서버의 호스트와 포트 번호에 대응되는 소켓 주소 구조체의 리스트를 반환**받는다.
    > 
    
    ```c
    		/* 
            Get a list of potential server addresses 
        */
        memset(&hints, 0, sizeof(struct addrinfo));
        hints.ai_socktype = SOCK_STREAM;  /* Open a connection */
        hints.ai_flags = AI_NUMERICSERV;  /* service인자를 포트 번호로 받는다. */
        hints.ai_flags |= AI_ADDRCONFIG;  /* Recommended for connections. |를 사용해 flag를 추가한다. */
        getaddrinfo(hostname, port, &hints, &listp);
    ```
    
- **적합한 클라이언트의 소켓을 만들고 연결을 성공시키기**
    
    > 서버의 hostname과 port에 대응하는 addrinfo 리스트를 돌면서 그 형식과 일치하는 소켓을 성공적으로 만들고, 서버와의 연결이 성공할 때까지 적합한 소켓 구조체를 찾는다.
    > 
    - socket()의 인자로 해당 서버의 소켓 주소 구조체의 형식과 맞는 소켓을 만든다.
    - connect()로 해당 소켓과 서버와의 연결을 시도한다.
    - 연결을 실패하면 소켓을 닫아주고 다음 소켓을 진행한다.
    
    ```c
    		for (p = listp; p; p = p->ai_next) {
            /* Create a socket descriptor */
            if ((clientfd = socket(p->ai_family, p->ai_socktype, 
                                   p->ai_protocol)) < 0)  // 식별자를 리턴 못하고 -1 리턴한다면.
                continue; /* Socket failed, try the next */
    
            /* Connect to the server */
            // socket()으로 만든 클라이언트 호스트의 소켓의 식별자, 서버의 주소를 넣고 서버와 연결을 시도한다.
            if (connect(clientfd, p->ai_addr, p->ai_addrlen) != -1)  //
                break; /* Success */
    
            Close(clientfd); /* 연결 실패하면 소켓 닫아주고 다음 소켓 진행 */
        }
    ```
    
- **addrinfo 리스트를 free하고 클라이언트의 소켓 식별자를 리턴한다.**
    
    ```c
    /* Clean up */
        freeaddrinfo(listp);
        if (!p) /* All connects failed */
            return -1;
        else    /* The last connect succeeded */
            return clientfd;
    }
    ```
    
<br>

**open_listenfd**
===

> 서버에서 연결 요청을 들을 준비가 된 듣기 식별자를 리턴한다. 즉 getaddrinfo(), socket(), bind(), listen()까지 해 준다.
> 

```c
#include "csapp.h"

int open_listenfd(char *port);
```

**파라미터**

- **해당 서버 프로세스의 서비스 종류, 즉 포트 번호 port**

**사용되는 함수**

> **addrinfo** 리스트 안의 구조체 멤버들은 모두 **바로바로 소켓 인터페이스 함수들의 인자로 사용**될 수 있다.
> 
- **socket ( 네트워크 주소 체계, 소켓 타입, 프로토콜 종류 )**
- **Setsocopt : 이전 바인딩에서 프로세스와 연결된 포트를 종료, 재시작, 연결 준비.**
- **bind ( 서버의 듣기 소켓의 식별자, 듣기 소켓과 맞는 서버의 소켓 주소, 주소의 길이 )**
- **listen ( 서버의 듣기 소켓의 식별자, backlog 인자 1024 )**

**리턴값**

- **클라이언트의 연결 요청을 받을 준비가 된 서버의 듣기 식별자**

<br>

**코드**

- **서버의 해당 서비스, 즉 포트 번호를 입력받는다.**
    
    ```c
    int open_listenfd(char *port)
    {
        struct addrinfo hints, *listp, *p;
        int listenfd, optval=1;
    ```
    
- **getaddrinfo() : 해당 포트와 연결할 수 있는 서버의 소켓 주소 리스트를 반환한다.**
    
    ```c
    		/* Get a list of potential server addresses */
        /* 해당 포트와 연결할 수 있는 서버의 소켓 주소 리스트를 반환한다. */
        memset(&hints, 0, sizeof(struct addrinfo));
        hints.ai_socktype = SOCK_STREAM;             /* Accept connect. TCP 프로토콜. */
        hints.ai_flags = AI_PASSIVE | AI_ADDRCONFIG; /* 듣기 소켓으로 && 내가 사용하는 IP 형식에 맞는 IP 주소만 */
        hints.ai_flags |= AI_NUMERICSERV;            /* service 인자를 port 번호만 받겠다. */
        
        /* 해당 port 번호에 대응하는 IP주소를 찾아보자. 
        듣기 소켓을 찾는 AI_PASSIVE flag를 설정했으므로 hostname은 NULL이다. */
        Getaddrinfo(NULL, port, &hints, &listp);
    ```
    
- **적합한 듣기 소켓을 만들고 와일드카드 주소와 바인드한다.**
    
    > 반환받은 듣기 소켓의 후보 소켓 주소 리스트를 탐색하면서 socket()으로 그 형식에 맞게 소켓을 만들고, 그 소켓과 와일드카드 주소를 bind()한다. bind가 성공하면 addrinfo 리스트를 free한다.
    > 
    
    ```c
    		/* Walk the list for one that we can bind to */
        for (p = listp; p; p = p->ai_next) {
            /* Create a socket descriptor */
            /* 해당 와일드카드 주소의 형식에 맞는 소켓 만들기 */
            if ((listenfd = socket(p->ai_family, p->ai_socktype, 
                                   p->ai_protocol)) < 0)
                continue;  /* Socket failed, try the next */
    
            /* 
            Eliminates "Address already in use" error from bind 
            이전 바인딩에서 포트가 프로세스와 연결되었으므로 
            해당 포트를 다시 쓰려면 포트(서버)를 재시작해야 한다. 30초정도 걸린다.
            */
            Setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, 
                       (const void *)&optval , sizeof(int));
    
            /* Bind the descriptor to the address */
            /* 우리가 만든 소켓과 우리가 찾은 와일드카드 주소와 바인드한다. */
            if (bind(listenfd, p->ai_addr, p->ai_addrlen) == 0)
                break; /* Success */
            Close(listenfd); /* Bind failed, try the next */
        }
    
    		/* Clean up */
        Freeaddrinfo(listp);
        if (!p) /* No address worked */
            return -1;
    ```
    
- **해당 소켓을 듣기 소켓으로 변환하고 그 소켓 식별자를 리턴한다.**
    
    ```c
    /* Make it a listening socket ready to accept conn. requests */
        if (listen(listenfd, LISTENQ) < 0) {
            Close(listenfd);
            return -1;
        }
        return listenfd;
    }
    ```