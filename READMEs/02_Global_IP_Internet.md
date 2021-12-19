**02. 글로벌 IP 인터넷**
===


> internet 중 하나로, internet을 가장 성공적으로 구현한 예시 중 하나다.
> 

<br>

**TCP/IP 프로토콜의 종류**
---

💡 각 인터넷 호스트는 TCP/IP 프로토콜을 구현한 소프트웨어를 사용한다.


- **IP**
    
    > 기본 명명법과 데이터그램이라는 패킷을 **호스트에서 호스트로** 전달하는 배달 메커니즘을 제공한다.
    > 
    
    데이터그램을 잃어버리거나 중복되는 경우에 아무것도 하지 않는다. 즉, 비안정적이다.
    
- **UDP**
    
    > IP를 더 확장해서, 데이터그램을 **프로세스에서 프로세스로** 배달하지만 역시 비안정적이다.
    > 
- **TCP**
    
    > IP를 더 확장해서 프로세스에서 프로세스로 **안전한 완전 양방향 연결**을 제공한다.
    > 
    - 흐름 제어
        
        보낸 데이터그램들이 순서대로 들어갈 수 있다.
        
    - 오류 제어
        
<br>

💡 호스트의 응용 프로그램(인터넷 클라이언트와 서버 등)은 **소켓 인터페이스**를 통해 TCP/IP 프로토콜을 이용한다.


<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146637274-6eca0d55-d717-493e-aecf-5977095416f6.png" width = "550">
</p>
<p align = 'center'>
<em>호스트 A에서 다른 LAN의 호스트 B로 데이터를 전송하는 과정.</em>
</p>

<br>

**IP 주소**
===

- **호스트**들은 각자 **고유의 32비트 IP 주소**에 맵핑된다. 32비트 주소를 이용하는 IP를 **IPv4**라 한다.
    - 예시 : Local host 127.0.1.1
    - IPv6는 128비트의 주소를 갖는다.
- 각 IP 주소들은 인터넷 도메인 네임이라는 식별자에 맵핑된다.
- 하나의 인터넷 호스트 안의 프로세스들은 다른 인터넷 호스트들과 **연결**을 통해 통신한다.

<br>

**in_addr struct**
---

> **32비트**의 IP 주소들은 **네트워크 바이트 순서(빅 엔디안)대로** IP 주소 **구조체에 저장**된다.
> 

💡 호스트들이 메모리에 바이트를 저장하는 방식**(호스트 바이트 순서)**이 서로 다를 수 있다. **IP 주소 구조체의 주소는** 호스트 바이트 순서에 상관없이 **늘 네트워크 바이트 순서로 저장**된다.

<br>

**Unix 운영체제에서 제공해주는 네트워크와 호스트 바이트 순서간의 변환 함수들**

> 64비트 값에 대한 함수는 없다! 필요하면 직접 구현.
> 

```python
#include <arpa/inet.h>

/* 호스트 -> 네트워크 */
uint32_t htol(uint32_t hostlong);
uint16_t htos(uint16_t hostshort);

/* 네트워크 -> 호스트 */
uint32_t ntol(uint32_t netlong);
uint16_t ntos(uint16_t netshort);
```
<br>

**Dotted Decimal Notation**
---

> IP 주소는 대개 dotted-decimal notation으로 제시된다. 각 바이트는 십진수 값을 사용하고, 다른 바이트와는 .으로 구분된다. **presentation 주소 버전**이라 부른다.
> 

**예시**

hex 주소  0x8002c2f2

변환        0x80.0x02.0xc2.0xf2

D.D.N      128.2.210.175

<br>

**변환 함수**

- **inet_pton** : Dotted Decimal string → 32비트 네트워크 바이트 순서 IP 주소
- **inet_ntop** : 32비트 네트워크 바이트 순서 IP 주소 → Dotted Decimal string

<br>

**Domain Naming System(DNS)**
---

> IP주소들을 도메인 이름에 매핑시키는 거대한 데이터베이스. 도메인 이름과 IP 주소를 서로 변환 시켜준다.
> 
- DNS 캐싱

    내가 최근에 방문했던 IP주소에 대한 정보가 메모리의 캐시에 저장되어 있다. 먼저 로컬 메모리의 캐시를 들러 저장되어있는 IP주소를 확인한다. 없으면 호스트 파일에서 확인한다. 그래도 없으면 그때서야 DNS 서버에서 IP주소를 요청한다.
- DNS 서버

    DNS 계층별로 DNS 서버를 따라가면서 해당 도메인 이름과 매칭되는 IP주소를 찾아준다.

- 루프백 주소 **Localhost 127.0.0.1**
    
    > 만약 네트워크 통신에서 패킷을 외부로 전송하지 않고 고스란히 내 호스트로 수신하고 싶을 때, 목적지 주소를 루프백 주소 127.0.0.1로 사용하면 된다.
    > 
- 도메인 이름은 하나의 IP 주소에 대해 여러 개를 가질 수 있다.
    
    ```c
    linux> nslookup cs.mit.edu
    Address: 18.62.1.6
    linux> nslookup eecs.mit.edu
    Address: 18.62.1.6
    ```
    
- 여러 개의 IP 주소에 대해 여러 개의 도메인 이름이 대응될 수도 있다.
    
    ```c
    linux> nslookup www.twitter.com
    Address: 199.16.156.6
    Address: 199.16.156.70
    Address: 199.16.156.102
    Address: 199.16.156.230
    
    linux> nslookup twitter.com
    Address: 199.16.156.102
    Address: 199.16.156.230
    Address: 199.16.156.6
    Address: 199.16.156.70
    ```
    
<br>

**인터넷 연결**
---

> 클라이언트와 서버는 **연결 connection**을 통해서 바이트 스트림을 주고받으며 통신한다.
> 

**소켓**

- **연결의 양 끝 점**을 의미한다.
- 소켓의 주소는 `IPaddress : port`로 표현한다.
- 연결 양 끝 점의 두 소켓을 **소켓 쌍**이라고 부르며, 튜플로 나타낼 수 있다.
    
    `( ClientAddress : ClientPort, ServerAddress : ServerPort )`
    

**포트**

- **IP 내에서 프로세스 구분**을 위한 **16비트 정수**이다. 각 프로토콜의 데이터는 논리적 포트를 통해 흐른다.
- **임시 포트 Ephemeral Port** : **클라이언트 쪽의 포트**는 연결 요청을 보낼 때 호스트의 커널이 자동으로 할당해준다.
- **잘 알려진 포트 well-known port** : 인터넷 내에서, **서버 쪽의 포트**는 대개 **서비스에 따라** 영구적으로 잘 알려진 포트들이 지정되어 있다. **각각 포트들은 이름이 있다.**
    
    ```c
    echo server: 7/echo
    ssh servers: 22/ssh
    email server: 25/smtp
    Web servers: 80/http
    ```
    
<Br>

**인터넷 연결의 구조**
---

> 인터넷 연결은 **연결의 양 끝 소켓 쌍에 의해 유일하게 정해진다.**
> 

클라이언트는 **호스트 커널에 의해 부여된 임시 포트**를, 서버는 **웹 서버이므로 80 포트**를 통해 데이터를 주고받는다.

<p align = 'center'>
<img src = "https://user-images.githubusercontent.com/93521799/146637276-70455ee4-c197-4774-b419-bf2b73e73566.png" width = "550">
</p>
<p align = 'center'>
<em>호스트 A에서 다른 LAN의 호스트 B로 데이터를 전송하는 과정.</em>
</p>
