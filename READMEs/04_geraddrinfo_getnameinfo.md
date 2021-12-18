**04. 호스트와 서비스 변환**
===
<br>

**getaddrinfo()**
===

> **모든 프로토콜**에 대해 **호스트 이름, 호스트 주소, 포트 번호, 서비스 이름을 소켓 주소 구조체로 변환**해준다. **모든 프로토콜에 대해!!!!!!!!**
> 

💡 **네트워크 호스트 정보**를 받아서 **네트워크 주소 정보(IP address)를 가져오거나 그 반대를 수행하는** 함수


- 멀티 쓰레딩 시 문제가 없다.
- IPv4나 IPv6 등 특정 IP 프로토콜에 의존할 필요가 없다.

<br>

💡 **입력한 host와 service(port)에 대응**되는 **addrinfo 구조체 링크드 리스트**를 가리키는 포인터를 반환한다.


```c

int getaddrinfo(const char *host,            /* Hostname or address */
                const char *service,         /* Port or service name */
                const struct addrinfo *hints,/* Input parameters */
                struct addrinfo **result);   /* Output linked list */

/* 메모리 누수를 피하기 위해 addrinfo 구조체 리스트를 반환한다. */
void freeaddrinfo(struct addrinfo *result);  /* Free linked list */

/* 에러 시에 에러 코드를 메세지로 바꿔준다. */
const char *gai_strerror(int errcode);
```
<br>

### **파라미터**

**host :** 도메인 주소 혹은 Dotted Decimal 주소 둘 다 가능

**service :** 서비스 이름(http....) 혹은 10진수 포트 번호(80, 22,...) 다 가능

<aside>
💡 host와 service 둘 중 하나를 인자로 넣으면 다른 하나는 NULL로 넣는다.

</aside>
<br>

**구조체 hint** : 내가 찾고 싶은 결과가 어떤 유형인지를 알려준다. addrinfo 구조체 result에 대해 희망하는 유형을 알려준다.

- `memset`을 이용해 우선 구조체를 **0으로 초기화**한 다음, 원하는 유형들을 선택한다.
- ai_family, ai_socktype, ai_protocol, ai_flags 등이 있다.
- **ai_family**
    
    IPv4 주소만 받거나(AF_INET), IPv6 주소만 받거나(AF_INET6), 둘 다 받을 수도(AF_UNSPEC) 있다.
    
- **ai_socktype**
    
    SOCK_STREAM으로 설정하면 이 소켓 주소를 연결 소켓으로 활용한다는 의미.
    
- **ai_protocol**
- **ai_flags**
    - **AI_ADDRCONFIG**
        
        로컬 호스트가 IPv4로 설정된 경우에**만** IPv4를 리턴한다. IPv6의 경우도 마찬가지. 만약 내가 IPv4 시스템을 사용하고 있다면 IPv6 주소를 받으면 에러가 날 것이다. 이를 사전에 방지한다.
        
    - **AI_CANONNAME**
        
        기본 값은 NULL이지만, 설정된다면 getaddrinfo 함수가 **addrinfo 구조체의 첫 리스트에 있는 호스트의 canonical(공식적) 이름을 가리키도록** 한다.
        
    - **AI_NUMERICSERV**
        
        service 인자가 서비스 이름일때는 받지 않는다. 오로지 포트 넘버만 받는다.
        
    - **AI_PASSIVE**
        
        기본적으로 getaddrinfo 함수는 클라이언트에 적합한 능동적인 소켓을 리턴한다. 만약 **이 flag가 켜지고 hostname이 NULL일 경우**에는 서버에서 사용할 수 있는 **듣기 소켓**의 주소, 즉 **와일드카드 주소**를 반환한다.
        
<br>

**사용 예시**
---

```c
#include <sys/types.h> 
#include <sys/socket.h> 
#include <netdb.h> 
int main(int argc,char *argv[]) 
{ 
		int status; 
		struct addrinfo hints; 
		struct addrinfo *servinfo; // 결과를 저장할 변수 
		memset(&hints, 0, sizeof(hints)); // hints 구조체의 모든 값을 0으로 초기화 
		hints.ai_family = AF_UNSPEC; // IPv4와 IPv6 상관하지 않고 결과를 모두 받겠다 
		hints.ai_socktype = SOCK_STREAM; // TCP stream sockets 
		
		status = getaddrinfo("www.google.com", "80", &hints, &servinfo); 
}

출처: https://techlog.gurucat.net/293 [하얀쿠아의 이것저것 만들기 Blog]
```
<br>

**addrinfo 구조체 리스트**
---

> 네트워크 주소정보(인터넷 주소)와 호스트이름을 표현하는데 사용된다.
> 

💡 IP주소는 굉장히 많은 유형을 지닌다. 하나의 도메인 이름에 여러 IP 주소가 맵핑될수도 있고, 하나의 IP 주소에 여러 도메인 이름이 맵핑될 수도 있다.

<br>

클라이언트는 이 리스트를 쭉 방문하면서 **각각의 소켓 주소를 socket()과 connect()가 성공할 때까지 계속 테스트**한다. 서버는 **socket()과 bind()가 성공할 때까지** 계속 테스트한다. 테스트에 성공하면 result에 그 소켓 주소 구조체를 가리키는 포인터를 리턴한다.

<br>

```c
struct addrinfo {
    int              ai_flags;     /* Hints argument flags */
    int              ai_family;    /* First arg to socket function */
    int              ai_socktype;  /* Second arg to socket function */
    int              ai_protocol;  /* Third arg to socket function  */
    char            *ai_canonname; /* Canonical host name */
    size_t           ai_addrlen;   /* Size of ai_addr struct */
    struct sockaddr *ai_addr;      /* Ptr to socket address structure */
    struct addrinfo *ai_next;      /* Ptr to next item in linked list */
};
```
<br>

**getaddrinfo()가 리턴하는 addrinfo 구조체 중 몇몇은 바로 소켓 인터페이스 함수에 인자로 들어갈 수 있다. 프로그래머가 걱정할 필요가 없다!**

- ai_family, ai_socktype과 ai_protocol는 socket()의 인자로 바로 들어갈 수 있고
- ai_addr와 ai_addrlen은 connect()와 bind()로 바로 들어갈 수 있다.

<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146637924-ad1d56f7-a100-444a-9ba0-f0c60f8b436e.png" width = "500">
</p>
<p align = 'center'>
<em>addrinfo() 구조체 리스트.</em>
</p>


<br>

**getnameinfo**
===

> getaddrinfo의 반대로, 네트워크 주소 정보(소켓 주소 구조체, IP주소)를 받아 그에 대응하는 호스트와 서비스 이름을(도메인 주소) 반환해준다.
> 

32비트 IP 주소를 입력하면 dotted-decimal 주소나 도메인 이름을 반환한다.

```c
int getnameinfo(const SA *sa, socklen_t salen, /* In: socket addr */
                char *host, size_t hostlen,    /* Out: host */
                char *serv, size_t servlen,    /* Out: service */
                int flags);                    /* optional flags */
```

<aside>
💡 소켓 주소 구조체 `sa`를 받아 이에 대응하는 호스트와 서비스 이름 스트링을 각각 `host`와 `serv` 버퍼에 복사한다.

</aside>

- **flags**
    - **NI_NUMERICHOST**
        
        도메인 이름을 리턴하지 않고 10진수 주소 스트링을 대신 리턴한다.
        
    - **NI_NUMERICSERV**
        
        default는 서비스 이름을 리턴하는 것이지만, 이 flag를 사용하며 포트 번호를 리턴한다.
        
<br>

**HOSTINFO 프로그램**
===

> 도메인 이름을 main()의 인자로 넣었을 때 IP주소를 반환한다.
> 

```c
#include "csapp.h"

int main(int argc, char **argv)
{
    struct addrinfo *p, *listp, hints;
    char buf[MAXLINE];
    int rc, flags;
```

**getaddrinfo()를 사용해 addrinfo 리스트를 받는다.**

```c
    /* 
			<Get a list of addrinfo records>
      getaddrinfo()로 해당 도메인에 대한 소켓 주소의 리스트 addrinfo를 반환한다.
      입력받은 main()의 인자 중 2번째(1번째는 실행파일명)의 도메인 주소로부터, 
      서비스는 도메인 이름만을 변환하기 위해 NULL, 힌트 추가, listp 반환
    */

    /* hint 구조체 세팅 */
    memset(&hints, 0, sizeof(struct addrinfo));  // hint 구조체 초기화
    hints.ai_family = AF_INET;       /* IPv4의 주소만 반환해주세요 */
    hints.ai_socktype = SOCK_STREAM; /* TCP 연결 */

    if ((rc = getaddrinfo(argv[1], NULL, &hints, &listp)) != 0) {
        fprintf(stderr, "getaddrinfo error: %s\n", gai_strerror(rc));
        exit(1);
    }
```

**getnameinfo()를 이용해 addrinfo 리스트에 있는 IP 주소들을 출력한다.**

```c
    /*
      addrinfo 리스트 하나하나 살펴보면서 ai_addr, 즉 IP주소를 반환한다. 
      domain은 getaddrinfo에서 받았고, 
      해당 domain에 대응되는 많은 IP주소들이 addrinfo 리스트에 나온다.
      각각의 IP주소를 domain으로 변환시키지 않고 그냥 출력한다.
    */
    flags = NI_NUMERICHOST; /* 도메인 이름을 리턴하지 않고 10진수 주소 스트링을 대신 리턴한다. */
    for (p = listp; p; p = p->ai_next) {
        getnameinfo(p->ai_addr, p->ai_addrlen,  // addrinfo 안의 IP주소(소켓 주소 구조체)를 찾아
                    buf, MAXLINE,   // 호스트 이름. flag를 썼으니 10진수 주소로.
                    NULL, 0,        // service는 안받아오는듯
                    flags);
        printf("%s\n", buf);  // input IP주소를 출력한다.

    }

    /* addrinfo 구조체를 free한다. */
    freeaddrinfo(listp);

    exit(0);
}
```

**결과**

```c
입력 ./hostinfo naver.com
223.130.195.200
223.130.195.95
223.130.200.107
223.130.200.104
```