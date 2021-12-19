**HTTP, Hypertext Transfer Protocol**
===

> 웹에서 **웹 클라이언트(브라우저)와 웹 서버가 서로 정보를 주고받을** 수 있는 응용 계층의 **프로토콜** 중 하나로, **TCP/IP 프로토콜을 이용**한다.
>

💡 HTTP는 이미지, 텍스트, 오디오 등 **여러 가지 유형의 데이터들을 주고받을 수 있다.** 이런 웹 컨텐츠는 **MIME 타입**으로 인코딩된다.

<br>

**MIME 타입, Multipurpose Internet Mail Extensions**
---
> 웹 서버에서 전송되는 여러 종류의 컨텐츠 유형들을 전달하기 위해 필요한 매커니즘이다. 

**역할**

- **바이너리 파일의 송수신**

    웹은 텍스트 파일 뿐만 아니라 여러 바이너리 파일(오디오, 이미지, 비디오 등)을 전송하기도 한다. 이런 바이너리 파일들을 문제 없이 전달하기 위해 파일을 **MIME 타입 텍스트파일로 인코딩**한다. 

- **데이터의 유형 명시(Content-type)**

    클라이언트와 서버는 주고받는 데이터 본체의 MIME 타입을 통해 데이터가 어떤 유형인지 파악하고, **어떻게 데이터를 해석할지를 결정한다.**
    
    예를 들어, 서버에서 클라이언트에게 준 데이터 본체의 MIME 타입이 `text/html`이라면, 클라이언트는 이 데이터가 `html` 형식의 텍스트 데이터라는 것을 알 수 있다.

    <p align = 'center'>
        <img src = 'https://user-images.githubusercontent.com/93521799/146664684-5f8f0582-a608-45e7-a8cc-89094a4845c5.PNG' width = '300'>
    </p>
    <p align = 'center'>
        <em>MIME 타입의 예</em>
    </p>

**Content-Type**

**HTTP 요청, 응답 메세지**의 헤더 중에는 **MIME 타입으로 표현된 Content-Type 헤더**라는 것이 있다. 이 HTTP 메세지의 바디 데이터를 상대가 어떤 MIME 타입으로 해석해야 할 지를 알려준다. 

<br>

**정적 컨텐츠, 동적 컨텐츠**
---
> **웹에서 컨텐츠는 단지 파일**이다. 웹 서버가 클라이언트에게 컨텐츠를 제공하는 유형을 크게 정적, 동적 컨텐츠로 나눌 수 있다.

- **정적 컨텐츠** : 변화가 없는 컨텐츠. 미리 서버의 파일 시스템에 저장해 둔 내용을 클라이언트에 상관 없이 응답만 한다.
- **동적 컨텐츠** : 실행파일이 런타임에 만든 출력을 클라이언트에게 보낼 때, 그 출력을 동적 컨텐츠라고 한다. 다시 말해 클라이언트의 요청에 따라 결과값이 달라지는 컨텐츠를 의미한다.

<br>

**URL, Uniform Resource Locator**
--
> 웹 네트워크 상에 있는 컨텐츠(자원)가 어디 있는지를 알려주는 주소이다. 작게는 웹 페이지, 크게는 컴퓨터 네트워크 상의 모든 자원의 위치를 나타낼 수 있다.


해당 웹 자원의 주소에 접근하려면 해당 URL에 맞는 프로토콜로 접속해야 한다.

```
http://www.example.com:80/path/to/myfile.html?key1=value1&key2=value2
```
**URL의 prefix**
- 프로토콜 : **http**

    어떤 프로토콜을 사용하여 데이터를 교환할 것인가? ex. HTTP, SSH, FTP 등등...
- 도메인 이름 : **www.example.com**
    
    웹 서버가 어디에 있는가?
- 포트 번호 : **:80**

    이 서버가 어느 포트를 듣고 있는가?

    각 프로토콜에 대한 표준 포트를 사용할 시에는 생략 가능. HTTP 프로토콜의 표준 포트는 80이다.

<br>

**URL의 suffix**

- 웹 서버 안에서 자원의 경로 : **/path/to/myfile.html**

    이 파일은 인터넷 호스트 www.example.com 안의 해당 경로에 있다. 그리고 HTML 유형이다.

- 웹 서버에 제공하는 추가 파라미터 **?key1=value1&key2=value2**

- 해당 URL이 **정적 or 동적 컨텐츠인지**
    
    특별한 방법은 없고, 만약 한 디렉토리를 만들어서 모든 동적 컨텐츠가 그 디렉토리에 저장되도록 할 수 있다(예. cgi-bin 디렉토리)

<br> 

**HTTP 트랜젝션**
===
**요청과 응답**
---

> 웹 클라이언트와 서버는 **요청과 응답**의 형태로 리소스를 주고받는다. 이런 요청과 응답은 **HTTP 메세지**라는 형식을 통해 이루어진다.

<p align = 'center'>
    <img src = 'https://user-images.githubusercontent.com/93521799/146671365-8a33433b-edc0-41c4-a04a-014944090650.PNG' width = '500'>
</p>
<p align = 'center'>
    <em>HTTP 요청과 응답</em>
</p>



**HTTP 요청**
---
**요청 라인**과 **요청 헤더**로 나뉜다.

**요청 라인**

> `<method> <uri> <version>` 의 형태를 가진다.

- **method**
    
    **서버가 어떤 동작을 해야 하는지**를 정해준다.

    GET, POST 등 여러 가지가 있지만 우선 GET만을 사용하기로 한다.
    
    **GET 메서드는 서버에서 클라이언트로 지정한 리소스를 보내라**는 뜻이다.
- **uri**

    브라우저가 컨텐츠를 요청할 때는 **해당 리소스의 URL 뒤의 suffix가 uri**이다. 

- **version**
    
    **요청이 어떤 HTTP 버전으로 포맷되어 있을지**를 말해준다.

<br>

**요청 헤더**

- **host 헤더**
    
    **꼭 있어야 하는 헤더**이다.

    같은 IP를 쓰는 하나의 서버 안에서 여러 사이트(도메인)들이 있을 수 있기 때문이다. 따라서 서버의 도메인 네임(포트 포함)을 명시해 주어야 한다.

    이를 [가상 호스팅](https://ko.wikipedia.org/wiki/%EA%B0%80%EC%83%81_%ED%98%B8%EC%8A%A4%ED%8C%85)이라고 한다. 

<br>


## **HTTP 응답** ##
---

**응답 라인**과 **응답 헤더**로 나뉜다.

### **응답 라인** ###

> `<version> <status code> <status message>` 의 형태를 가진다.

- **version**
    
    **응답의 HTTP 버전**을 정해준다.

- **status code 상태 코드**
    
    요청이 성공했는지, 다른 조치가 필요한지 등의 **요청 특성**을 알려준다.

- **status message**
    
    **에러 코드를 영어**로 알려준다.

<br>

### **응답 헤더** ###

- **Content-Type**

    **응답 본체(body)** 내 **컨텐츠의 MIME 타입**을 알려준다.

- **Content-length**

    **응답 본체(body)** 내 **컨텐츠의 길이**를 알려준다.

<br>

**Telnet을 이용한 HTTP 트랜젝션**
---


Telnet 프로그램을 통해 인터넷 상의 모든 웹 서버와 간단하게 트랜젝션해보자.

- 서버 연결
    ```
    whaleshark> telnet www.cmu.edu 80          Client: open connection to server 
    Trying 128.2.42.52...                      Telnet prints 3 lines to terminal
    Connected to WWW-CMU-PROD-VIP.ANDREW.cmu.edu.
    Escape character is '^]'.
    ```
<br>

- **HTTP 요청 : 클라이언트**

    ```
    GET / HTTP/1.1                             요청 라인. HTTP 1.1 버전으로 요청을 보낼 것이다.
    Host: www.cmu.edu                          required HTTP/1.1 header. Host 헤더 사용
                                               empty line terminates headers. 요청 헤더 종료.
    ```
<br>

- **HTTP 응답 : 서버**

    ```
    HTTP/1.1 301 Moved Permanently             응답 라인 response line
    Date: Wed, 05 Nov 2014 17:05:11 GMT        응답 헤더 1.
    Server: Apache/1.3.42 (Unix)               응답 헤더 2. this is an Apache server
    Location: http://www.cmu.edu/index.shtml   응답 헤더 3. page has moved here. 이 페이지로 넘어갔다.
    Transfer-Encoding: chunked                 응답 헤더 4. response body will be chunked
    Content-Type: text/html; charset=...       응답 헤더 5. expect HTML in response body(MIME!)
                                               empty line terminates headers
    15c                                        응답 본체 시작
    <HTML><HEAD>                               start of HTML content
    …
    </BODY></HTML>                             end of HTML content
    0                                          last line in response body
    Connection closed by foreign host.         closes connection

    ```

<br>

# **동적 컨텐츠의 처리** #

> 클라이언트가 요청을 보낼 때 어떤 인자를 서버에 같이 보내면, 서버가 그 인자에 맞는 응답을 줄 수 있다. 이게 동적 컨텐츠이다.

근데 어떻게 클라이언트가 요청을 주는 대로 서버가 때에 따라 다른 응답을 줄 수 있을까?

<br>

## **CGI, Common Gateway Interface** ##
---

> CGI는 웹 서버에서 동적인 페이지를 보여주기 위한 **임의의 자식 프로세스**를 실행할 수 있게 해 준다.

원래 웹 서버는 서버에 저장되어 있는 고정된 문서를 보여준다(정적 컨텐츠). 

하지만 데이터베이스 조회, 정보 기록, 로직 처리 등의 요청을 수행하기 위해서는 웹 서버만으로는 할 수 없었다. 

대신 웹 서버가 **특정 URL로 들어가** 클라이언트의 **요청을 특정한 프로그램에 넘겨**줘서 이런 작업들을 수행할 수 있게 되었는데, 이 기술이 바로 **CGI**이다.



<p align = 'center'>
    <img src = 'https://user-images.githubusercontent.com/93521799/146671271-11f0debb-2294-4993-9470-9c7813075fde.png' width = '450'>
</p>
<p align = 'center'>
    <em>CGI 구동 원리.</em>
</p>

<br>

## **작동 방식** ##
---

### **클라이언트가 서버에 프로그램 인자를 전달하는 방법** ###

> GET 요청을 위한 인자들은 URI의 `?`과 `&`로 전달된다.

요청 라인에 `GET /cgi-bin/adder?15000&213 HTTP/1.1` 과 같은 형태로.

<br>

### **서버가 이 인자들을 임의의 프로세스에 전달하는 방법** ###

1. 요청을 받으면 fork() 시스템 콜을 통해 자식 프로세스를 생성.
2. 자식 프로세스가 [CGI 환경변수](https://www.ibm.com/docs/ko/netcoolomnibus/8.1?topic=SSSHTQ_8.1.0/com.ibm.netcool_OMNIbus.doc_8.1.0/webtop/wip/reference/web_cust_envvariablesincgiscripts.html) QUERY_STRING을 인자 `"15000&213"`으로 설정
3. 서버가 execve() 시스템 콜을 호출해 /cgi-bin/adder 프로그램을 자식 프로세스의 컨텍스트에서 실행.
4. adder 프로그램이 런타임에 `getenv()`를 써서 그 환경 변수의 값을 참조한다.

<br>

### **서버가 자식 프로세스에 정보를 전달하는 방법** ###

> 자식 프로그램이 실행될 때 환경 변수에 값이 지정되어 있어야 한다.

<br>

### **자식 프로세스가 출력을 내보내는 방법** ###

> CGI 프로그램은 자신의 컨텐츠를 표준 출력으로 내보낸다.

1. `dup2` 함수를 이용해 **CGI 프로세스의 표준 출력으로** 클라이언트와 연결된 **서버의 소켓 연결 식별자를 지정**한다.

    표준 출력으로 CGI가 쓰는 것들이 모두 클라이언트로 바로 가게 된다!
<br>

2. 자식 프로세스가 응답 헤더의 `Content-type`과 `Content-length`, `헤더 종료 빈 칸`까지 만들어준다.

<br>

##