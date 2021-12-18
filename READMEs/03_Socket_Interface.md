**03. 소켓 인터페이스**
===
<br>

**소켓**
===

**소켓**
---

**커널의 관점**

> 통신의 양 끝 점이다.
> 

**응용 프로그램의 관점**

> 네트워크를 포함한 모든 Unix I/O 디바이스들은 파일이므로, 소켓 역시 네트워크 상의 다른 프로세스와 통신하는 역할을 하는 파일로 볼 수 있다.
> 

💡 **클라이언트와 서버는 소켓 식별자를 읽고 쓰면서 서로 통신한다.**

<br>

**소켓 주소 구조체**
--

💡 소켓 주소 = IP주소 + 포트 번호

<br>

**generic 소켓 주소 구조체 sockaddr**
---

> connect, bind, accept 함수의 인자로 넣어주기 위한 16바이트 구조체이다.
> 

예전에는 C언어에 (void *) 타입이 없어서 인자를 캐스팅해주기 위해 필요했다.

```c
struct sockaddr { 
  uint16_t  sa_family;    /* Protocol family */ 
  char      sa_data[14];  /* Address data.  */ 
};
```

- **sa_family 필드** : 이 **소켓의 종류가 무엇인지**를 말해준다. TCP인지, UDP인지, IPv6인지.

<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146637610-7b46a4af-bf4a-4152-b00a-0e684df3d916.png" width = "550">
</p>
<p align = 'center'>
<em>generic 소켓 주소 구조체</em>
</p>


<br>

**IPv4 인터넷 소켓 주소 구조체 sockaddr_in**
---

> Internet에서 활용되는 인터넷 소켓 주소는 더 구체적인 정보를 담고 있는 16바이트 구조체에 저장된다. IPv4 소켓에서 사용되는 종류이다.
> 

<aside>
💡 소켓 주소 구조체 sockaddr를 인자로 받는 함수에 넣어주기 위해서는 (struct sockaddr *)으로 형변환 해주어야 한다.

</aside>

```c
struct sockaddr_in  { 
  uint16_t        sin_family;  /* Protocol family (always AF_INET) */ 
  uint16_t        sin_port;    /* Port num in network byte order */ 
  struct in_addr  sin_addr;    /* IP addr in network byte order */ 
  unsigned char   sin_zero[8]; /* Pad to sizeof(struct sockaddr) */ 
};
```

- **sin_family 필드** : AF_INET(32비트 IPv4 주소)를 사용한다는 것을 나타냄. **항상 AF_INET**이다.
- **sin_port 필드** : **16비트 port 번호**. **네트워크 바이트 순서**로 저장된다.
- **sin_addr 필드** : **32비트 IP 주소**. **네트워크 바이트 순서**로 저장된다.
- **sin_zero 필드** : sockaddr 구조체와 사이즈를 맞추기 위한 패딩.

<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146637611-d04c7304-4a81-4654-a52d-6f59a9d3f093.png" width = "550">
</p>
<p align = 'center'>
<em>인터넷 소켓 주소 구조체 sockaddr_in</em>
</p>

<br>

**소켓 인터페이스**
---

> TCP/IP 계층의 응용 계층에서 전송 계층의 기능을 사용할 수 있도록 제공하는 응용 프로그래밍 인터페이스 (API, Application Programming Interface). 소켓 인터페이스는 **응용 프로그램과 TCP 계층을 연결하는 역할**을 한다.
> 

💡 TCP/IP 프로토콜을 이용해 응용 프로그램을 작성할 때 TCP 계층에서 제공하는 인터페이스 함수를 직접 사용해 프로그래밍하면 매우 복잡하다. 소켓 인터페이스를 이용하면 **전송 계층이나 네트워크 계층의 복잡한 구조를 몰라도 쉽게 네트워크 프로그램을 작성할 수 있다.**

<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146637612-c9cb4cc5-79aa-415e-8f24-e89a90b4a526.png" width = "400">
</p>
<p align = 'center'>
<em>소켓 인터페이스를 활용한 통신 구조</em>
</p>


<br>

**소켓 인터페이스 기반 네트워크 응용프로그램**
---

> 클라이언트와 서버가 요청을 송/수신하고 요청을 처리하는 과정이 어떻게 이루어지는가?
> 

<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146637615-8c6120e2-a7cb-4e1d-b22e-58d0b6cc88d7.png" width = "500">
</p>
<p align = 'center'>
<em></em>
</p>


1. **Start Server**
    
    > 서버 위의 프로그램이 요청을 받아 수행하기 위해 서버가 셋업되는 과정. 클라이언트보다 서버가 먼저 있어야지!
    > 
2. **Start Client**
3. **Exchange Data**
    
    > 클라이언트/서버 **세션에서** 서로 데이터를 교환한다.
    > 
    
    RIO(Reliable I/O)를 사용하여 읽기 쓰기 작업을 수행한다.
    
4. **Disconnect Client**
5. **Drop Client**
    
    > 서버가 클라이언트를 드랍하고 새 클라이언트 요청을 받는다.
    > 
    
<br>

**socket()**
---

> 클라이언트와 서버가 **소켓(소켓 판별자)를 만들기 위해 사용**하는 함수.
> 

```c
int socket(int domain, int type, int protocol)

예시
int clientfd = Socket(AF_INET, SOCK_STREAM, 0);
```

**파라미터**

- **네트워크 주소 체계 domain: AF_INET**
    
    32비트 IPv4 주소를 사용한다는 것을 알려준다.
    
- **소켓 타입 type: SOCK_STREAM**
    
    우리가 만들 소켓의 타입이 TCP 프로토콜을 사용한 통신의 소켓이라는 뜻.
    

**리턴값**

- 파일을 대신할 수 있는 **식별자(작은 정수)를 리턴**한다.
- 실패하면 -1

<br>

**bind(), 서버 함수**
---

> 서버의 소켓 주소와 프로세스가 만든 소켓을 서로 묶는다. 커널 콜이다. 즉, 커널이 이 역할을 한다.
> 

💡 응용 프로그램이 socket()을 이용해 소켓을 만들면, 통신에 이 소켓을 사용하기 위해 **소켓의 네트워크 시스템(TCP/IP) 주소를 그 소켓에 붙여주어야 한다.** 그래야 응용 프로세스와 네트워크 시스템 간의 패킷 전달이 가능하다.


```c
int bind(int sockfd, sockaddr* addr, socklen_t addrlen);
```

**파라미터**

- **서버의 응용 프로그램이 만든 소켓의 번호(식별자) sockfd**
- **서버의 소켓 주소 (sockaddr *) addr**
    
    소켓의 다른 한쪽 끝(연결의 끝)의 주소
    
- **주소의 길이 addrlen**
    
    IPv4의 경우는 4이다(바이트).
    

**리턴값**

- 성공했으면 0, 아님 -1.

**클라이언트는 왜 bind()가 필요가 없을까?**

- 클라이언트는 포트 번호를 임의로 사용해도 된다. 즉, IP 주소나 포트번호를 다른 클라이언트 또는 서버가 미리 알고 있을 필요가 없다.
- 서버의 응용 프로그램은 자신이 사용하고 있는 포트 번호를 통하여 클라이언트들의 서비스를 처리해야 하므로, 응용 프로그램이 소켓 번호와 소켓 주소를 bind()하는 것이 필수적이다.

<br>

**listen(), 서버 함수**
---

> 해당 소켓이 **서버 소켓임을 알려준다.**
> 

socket()이 만든 소켓은 기본적으로 클라이언트 단의 능동적인 소켓이다. 즉, 연결을 요청하는 역할을 한다. 그에 비해 **서버 소켓은 클라이언트의 연결 요청을 기다리는 입장**이므로, 이렇게 수동적 소켓임을 알려주기 위해 사용한다.

```c
int listen(int sockfd, int backlog);
```

**파라미터**

- **서버의 응용 프로그램이 만든 소켓의 번호(식별자) sockfd**
- **커널이 요청을 거절하기 전에 큐에 저장해야 하는 연결의 수 backlog**
    
    일반적으로 1024라는 큰 수를 넣는다.
    

**리턴값**

- 성공했으면 0, 아님 -1.

<br>

**accept(), 서버 함수**
---

> 서버가 연결 준비가 되었다는 것을 알려주고 **연결 request를 받는다.**
> 

```c
int accept(int listenfd, SA *addr, int *addrlen);
```

**파라미터**

- **클라이언트의 연결 요청을 받는 듣기 소켓의 번호 listenfd**
- **클라이언트에게서 요청을 받고 나면 알게 되는 클라이언트의 소켓 주소 (sockaddr *) addr**
- **주소의 길이 addrlen**

**리턴값**

- **연결 식별자 connected descriptor**
    - 이를 통해 Unix I/O 함수들을 사용해서 클라이언트와 통신할 수 있다.
- 실패했으면 -1

<br>

**connect(), 클라이언트 함수**
---

> 서버에 연결 요청을 보내 **서버와의 연결을 수립**한다.
> 

```c
int connect(int clientfd, SA *addr, socklen_t addrlen);
```

**파라미터**

- **클라이언트의 소켓의 번호 clientfd**
- **서버의 소켓 주소 (sockaddr *) addr**
- **주소의 길이 addrlen**

**리턴값**

- 성공한다면 0, 실패했으면 -1

**결과**

- 성공한다면 clientfd 파일을 읽고 쓸 수 있게 된다.
- 성공한 연결은 `(x : y, addr.sin_addr : addr.sin_port)`와 같이 소켓 쌍으로 표현할 수 있다.
    - x : 클라이언트 주소
    - y : 클라이언트 호스트 내의 해당 프로세스를 식별하는 유일한 **포트 주소**

<br>

**듣기와 연결 식별자**
---

> 듣기 식별자는 오로지 다수의 **클라이언트로부터의 연결 요청만 받는 소켓**이다. 연결 식별자는 **각각의 클라이언트들과 해당 서버의 포트를 이어주어 데이터를 주고받게** 해 준다.
> 
- 듣기 식별자는 서버가 종료될 때까지 계속 살아있는다. 유일하게!
- **같은 서버 포트에 여러 개의 연결 식별자가 있을 수 있다.**
- 연결 식별자가 여러 개 존재함으로써 **서버 하나가 여러 개의 클라이언트들과의 연결을 동시에 진행**할 수 있다.

<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146637616-96fd990b-16ca-41f9-aca2-03cb2531580c.png" width = "500">
</p>
<p align = 'center'>
<em>듣기 식별자가 클라이언트의 connect()로부터 연결 요청을 받으면, 서버가 accept()를 통해 클라이언트와 연결 식별자를 이어준다.</em>
</p>


