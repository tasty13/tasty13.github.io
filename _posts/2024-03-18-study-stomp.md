---
layout: post
title: STOMP를 이용한 채팅 구현
subtitle: websocket, stomp
categories: [Study, Web]
---

SSE와 EventListener를 이용해서 알림을 구현할 때 웹소켓이랑 SSE 중 고민하다가 SSE를 사용했는데 채팅을 구현하게 되면서 양방향 통신을 위해 웹소켓에 대해 알아보았다.

## STOMP

웹소켓은 통신 프로토콜일 뿐이지 특정 주제를 구독하는 사용자에게만 메시지를 보내는 방법이나 특정 사용자에게 메시지를 보내는 방법 같은 걸 정의하지 않는다.<br/>
따라서 메시지 종류/형식/각 메시지 내용의 정의를 위해 STOMP를 사용해야 한다.


### 소개

STOMP는 프레임 기반 프로토콜로 프레임이 HTTP를 기반으로 모델링된다.<br/>
STOMP 프레임의 구조:
```
COMMAND
header1:value1
header2:value2

Body^@
```

클라이언트는 메시지 내용과 수신 대상자를 설명하는 destination 헤더와 함께 SEND 혹은 SUBSCRIBE 명령을 사용해 메시지를 보내거나 구독할 수 있다. 
이는 브로커를 통해 연결된 다른 클라이언트에 메시지를 보내거나 일부 작업을 수행하도록 요청하는 데 사용할 수 있는 간단한 게시-구독 메커니즘을 가능하게 한다.





STOMP를 사용하는 경우 Spring WebSocket App은 클라이언트에 대한 STOMP 브로커 역할을 한다. 이때 메시지는 @Controller 메시지 처리 메서드나 구독한 사용자에게 메시지를 브로드캐스트하는 Simple-Broker로 라우팅된다.
- Simple-Broker → 클라이언트의 구독 요청을 처리하여 메모리에 저장한 후 일치하는 목적지가 있는 연결된 클라이언트에 메시지를 브로드캐스트한다.


- PUB/SUB 구조 (메시지 공급 주체와 소비 주체를 분리해 제공하는 메시징 방법)
    - Publisher는 메시지를 우체통(Topic)에 넣음
    - Subscriber는 우체통에 있는 메시지를 꺼내 봄
        
→ 내가 구현해야 할 채팅 서비스의 관점에서 보면 채팅방이 우체통(Topic), 채팅방에 있는 사람들은 Subscriber, 채팅방에 메시지를 보내는 건 Publisher다.
        
### 메시지 전달 과정
1. 프론트는 서버의 컨트롤러에서 설정한 엔드포인트 연결
2. 프론트는 보내고 싶은 메시지를 집배원(/PUB)에게 publish
3. 프론트가 publish한 메시지를 받고 싶으면 MessageMapping한 /SUB 구독

<br/>
<br/>

이번에 구현한 내용은 웹소켓을 특정 경로로 연결해준 뒤 어떤 주소로 보내면 특정 채팅방을 구독하고 있는 사람에게 알림이 가는 형식이다. 이전의 채팅 내용은 GetMapping을 통해 가져온다. 앱으로 채팅 기능을 만들 때는 이상한 오류 때문에 진짜 엄청 삽질했던 것 같은데 웹으로 하니까 훨씬 쉽다… 테스트도 간단하게 끝났다…






... 이하 내용 추가예정




---
참고자료
- Spring 공식 문서
    - [STOMP](https://docs.spring.io/spring-framework/reference/web/websocket/stomp.html)
    - [STOMP-enable](https://docs.spring.io/spring-framework/reference/web/websocket/stomp/enable.html)
    - [Simple Broker](https://docs.spring.io/spring-framework/reference/web/websocket/stomp/handle-simple-broker.html)
    - [messaging-stomp-websocket](https://spring.io/guides/gs/messaging-stomp-websocket/)