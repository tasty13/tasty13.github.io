---
layout: post
title: 웹소켓이란?
subtitle: websocket, stomp
categories: [Study, Web]
---

## 들어가기 전에
### TCP/IP
: 네트워크와 네트워크를 연결하는 데 사용하는 프로토콜
- TCP: 전송 제어 프로토콜, IP: 인터넷 프로토콜

- TCP는 전송된 패킷에 오류가 발생하면 재전송 요청, 오류가 없으면 원래의 데이터로 재결합
<br/> -> 데이터의 흐름을 제어하거나 데이터가 정확한지 검사 (정확성 보장)

- IP는 패킷으로 변환된 데이터를 네트워크를 통해 멀리 떨어진 호스트에 보냄 (패킷 전송 역할)

### TCP handshake 
: 데이터가 전송되었는지 확인하면서 이루어지는 통신 방식 (= 3-way handshake = SYN-SYN-ACK)

![http_communication_1](https://github.com/tasty13/tasty13.github.io/assets/101929240/3a83aa6a-c047-4ce7-961c-054aad01b1ad)

1. 송신 측 컴퓨터는 수신 측 컴퓨터에 연결 확립 허가를 받기 위한 SYN 요청을 보낸다.
2. 수신 측 컴퓨터는 송신 측 컴퓨터가 보낸 요청을 받은 후 허가한다는 응답을 회신하기 위해 연결 확립 요청인 SYN을 보낸다. 이때 연결을 확립하기 위해 코드 비트의 SYN과 ACK가 1로 활성화된다. (SYN-ACK 보냄)
3. 수신 측 컴퓨터의 요청(SYN-ACK)을 받은 송신 측 컴퓨터는 수신 측 컴퓨터에 허가한다는 응답으로 연결 확립 응답인 ACK를 보낸다.

<br/>

---

## HTTP
### HTTP
: TCP 기반으로 만들어진 프로토콜
: 클라이언트와 웹 브라우저가 서버에 웹 서비스를 요청하면 서버가 적절한 응답을 하여 클라이언트의 사용자에게 웹 페이지를 제공하는 서비스<br/>
-> 서버와 클라이언트 간 하이퍼텍스트 문서를 송수신하는 프로토콜
- 동작 과정
![http_communication_1](https://github.com/tasty13/tasty13.github.io/assets/101929240/0a4a38e7-9c29-4df1-a98d-8f38def636b3)
    1. 클라이언트가 웹 브라우저에 URL 주소 입력
    2. TCP 포트 번호 80을 이용해 접속하려는 서버에 연결 시도
    3. 클라이언트는 TCP 요청 소켓을 이용해 URL 주소를 포함한 요청 메시지를 서버에 전송
    4. 서버는 클라이언트의 요청 메시지에 응답하여 소켓을 통해 메시지를 전송하고 TCP 연결 설정 헤제

<br/>
<br/>



### polling
![polling](https://github.com/tasty13/tasty13.github.io/assets/101929240/2ff2ea65-ebb1-498a-adc3-fafe4030817f)
- 클라이언트가 서버에게 일정한 주기로 요청을 보내고 응답을 받는다.
- 요청의 유무와 상관없이 주기마다 보내기 때문에 실시간성이 보장되지 않는다.
- 변경 사항이 없어도 지속적으로 서버에 요청을 보낸다.<br/>
=> 클라이언트의 수가 많을수록/요청을 보내는 주기가 짧아질수록 서버의 부담이 커진다.


### long-polling
![long-polling](https://github.com/tasty13/tasty13.github.io/assets/101929240/24efea42-9210-4241-8b25-86d0a85be29b)
- 클라이언트가 요청을 보내면 서버는 새 이벤트가 있거나 타임아웃이 발생한 경우에만 응답한다.
- 클라이언트가 응답을 받으면 즉시 서버에 새 요청을 보내고, 서버는 클라이언트에 데이터를 전송하기 위해 대기 중인 연결을 새로 설정한다.
- 이벤트들의 시간 간격이 짧다면 polling과 별 차이가 없다. (서버에서 보내는 메시지가 빈번하지 않은 경우 사용)

### streaming
![streaming](https://github.com/tasty13/tasty13.github.io/assets/101929240/2a6f4ae4-a849-4f3f-977a-3a8a9497a1a8)
- 클라이언트가 요청을 보내면 서버는 HTTP 연결을 끊지 않고 지속적으로 클라이언트에 이벤트를 보낸다.
- polling 방식과 달리 재연결에 대한 부하가 없다.

<br/>

---

## 웹소켓
### 웹소켓
: 단일 TCP 연결을 통해 클라이언트-서버 간 전이중 양방향 통신 채널을 구축할 수 있도록 하는 방법 제공

![websocket](https://github.com/tasty13/tasty13.github.io/assets/101929240/73bb08fd-cc1c-4e69-8fd1-912260343b8d)



### 특징

- 전이중 방식: 쌍방향 동시 통신
    - 반이중 방식: 두 장치 간 교대로 데이터 교환. 전이중과 반이중 방식 둘 다 양방향 통신이다.
    - TCP는 대표적인 전이중 전송방식
- TCP 프로토콜이나 포트 80(http 기본 포트)과 443(https 기본 포트)를 사용
- 기존 방화벽 규칙을 재사용할 수 있도록 설계되어 HTTP를 통해  작동

웹소켓은 TCP로 실행하기 때문에 지연 시간이 짧고, low-level 통신을 제공하며, 각 메시지의 오버헤드를 줄인다.
<br/>
핸드셰이크를 성공한 후에도 클라이언트와 서버 모두 메시지를 주고받을 수 있도록 TCP 소켓이 열려 있다.

### 사용처

실시간 통신이 필요할 때, 즉 짧은 지연 시간, 높은 빈도, 대용량이 요구될 때 웹소켓을 사용한다.








<br/>
<br/>
<br/>

참고자료
---
- 네트워크 개론 (진혜진, 한빛아카데미)
- [TCP handshake](https://developer.mozilla.org/ko/docs/Glossary/TCP_handshake)
- [A Guide to the Java API for WebSocket](https://www.baeldung.com/java-websockets)
- [javascript.info - websocket](https://ko.javascript.info/websocket)
- [spring.io - WebSockets](https://docs.spring.io/spring-framework/reference/web/websocket.html#)
- [Polling & Long Polling & Streaming](https://dydtjr1128.github.io/etc/2019/09/23/polling-long-polling-streaming.html)
- [Socket.IO (1/4)](https://bcho.tistory.com/896)
- [웹 브라우저에서의 통신 방법](https://warmth424.tistory.com/18)