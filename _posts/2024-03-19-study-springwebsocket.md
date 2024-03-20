---
layout: post
title: 웹소켓 사용해보기
subtitle: websocket, stomp
categories: [Study, Web]
---

## STOMP

웹소켓은 통신 프로토콜일 뿐이지 특정 주제를 구독하는 사용자에게만 메시지를 보내는 방법이나 특정 사용자에게 메시지를 보내는 방법 같은 걸 정의하지 않는다.<br/>
따라서 메시지 종류/형식/각 메시지 내용의 정의를 위해 STOMP를 사용해야 한다.


### 소개

STOMP는 프레임 기반 프로토콜로 프레임이 HTTP를 기반으로 모델링된다.

STOMP를 사용하는 경우 Spring WebSocket App은 클라이언트에 대한 STOMP 브로커 역할을 한다. 이때 메시지는 @Controller 메시지 처리 메서드나 구독한 사용자에게 메시지를 브로드캐스트하는 Simple-Broker로 라우팅된다.

Simple-Broker란?<br/>
→ 클라이언트의 구독 요청을 처리하여 메모리에 저장한 후 일치하는 목적지가 있는 연결된 클라이언트에 메시지를 브로드캐스트(모든 수신자에게 동시에 메시지를 전송)

### 구조
- PUB/SUB 구조 (메시지 공급 주체와 소비 주체를 분리해 제공하는 메시징 방법)
    - Publisher는 메시지를 우체통(Topic)에 넣음
    - Subscriber는 우체통에 있는 메시지를 꺼내 봄
        
→ 내가 구현해야 할 채팅 서비스의 관점에서 보면 채팅방이 우체통(Topic), 채팅방에 있는 사람들은 Subscriber, 채팅방에 메시지를 보내는 건 Publisher다.
        
### 메시지 전달 과정
1. 프론트는 서버의 컨트롤러에서 설정한 엔드포인트 연결
2. 프론트는 보내고 싶은 메시지를 집배원(/PUB)에게 publish
3. 프론트가 publish한 메시지를 받고 싶으면 MessageMapping한 /SUB 구독


## 구현 전 테스트
[Using WebSocket to build an interactive web application](https://spring.io/guides/gs/messaging-stomp-websocket)
스프링 공식 사이트의 문서를 참고해서 각각의 코드가 어떤 역할을 하는지 분석해보았다.

BACKEND
---


### GreetingController

``` java
@MessageMapping("/hello")
@SendTo("/topic/greetings")
public Greeting greeting(HelloMessage message) throws Exception {
    Thread.sleep(1000); // simulated delay
    return new Greeting("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!");
}
```
Controller의 greeting 메서드는 파라미터로 HelloMessage(이름) 받음
-> Greeting 객체 만들어서 리턴하는 형태. 이때 sendTo로 리턴해줌


@MessageMapping 어노테이션-> 메시지가 목적지인 /hello에 전송되면 greeting() 메서드가 호출되도록 함

메시지의 페이로드는 greeting() 메서드로 전달되는 HelloMessage 객체에 바인딩됨
여기서 페이로드란 전송하고자 하는 실제 데이터

여기서 스레드를 1초 동안 절전 모드로 전환해서 처리를 지연했는데 얘는 클라이언트가 메시지를 보낸 후에 서버가 비동기적으로 메시지를 처리하는 데 필요한 만큼 시간이 걸릴 수 있다는 걸 보여준거임 클라이언트는 응답 기다릴 필요 없이 작업 계속할 수 있음
-> 뭐... 구현에는 딱히 의미없는 부분이네요 알앗어요

greeting() 메서드는 Greeting 객체를 생성하고 이를 반환하는데 이때 반환 값은 @SendTo 어노테이션에 지정된 대로 /topic/greetings의 모든 구독자에게 브로드캐스트됨

이 경우 입력 메시지는 클라이언트 측의 브라우저 DOM에서 echoed back and re-rendered되므로 입력 메시지의 이름은 sanitize된다
- echoed back
    - 데이터 통신에서 네트워크를 거친 데이터 전송시 송신측과 수신측에서 올바로 송수신되었는지를 탐지하기 위해 수신측이 송신측으로 데이터가 반송한다. 
이 때 반송하는 것을 에코백(echo back)이라 하고, 반송된 데이터를 에코(echo)라고 한다.
- sanitize-> XSS 방지하는 모듈
    - html의 input 또는 textarea 또는 기타등등의 사용자 입력정보에 <script>문자</script> 이란 문자열을 적을시, 웹브라우저에서 문자열이 txt가 아닌 script 기술로 받아들여서 생기는 문제를 방지
    - 사용자가 이를 악용하여 `<script>`,`<a>` 등등 기타 태그들을 삽입해서 악성 스크립트로 변질시켜 실행시키는 것 방지

<br/>

### WebSocketConfig

``` java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

  @Override
  public void configureMessageBroker(MessageBrokerRegistry config) {
    config.enableSimpleBroker("/topic");
    config.setApplicationDestinationPrefixes("/app");
  }

  @Override
  public void registerStompEndpoints(StompEndpointRegistry registry) {
    registry.addEndpoint("/gs-guide-websocket");
  }
}
```

- `@Configuration` -> 스프링 구성 클래스

- `@EnableWebSocketMessageBroker`->  메시지 브로커라는 걸 통해 WebSocket 메시지 처리를 활성화

- `configureMessageBroker()`-> 메시지 브로커를 구성하기 위해 WebSocketMessageBrokerConfigurer의 기본 메서드를 구현
    - `enableSimpleBroker("/topic")`
        - 메시지 브로커가 /topic으로 시작하는 주소의 구독자들에게 메시지를 전달하도록 함
        - /topic은 한 명이 msg 발행했을 때 해당 토픽을 구독하고 있는 n명에게 메시지를 뿌려야 할 때 사용
        - /queue는 한 명이 msg 발행했을 떄 발행한 한 명에게 다시 정보를 보내는 경우
    - `setApplicationDestinationPrefixes("/app")`
        - 클라이언트가 서버로 메시지를 발송할 수 있는 경로의 prefix 지정
        - Controller에서 /chat이라는 토픽에 대해 구독을 신청했을 때 실제 경로: /app/chat

- `registerStompEndpoints()`
    - 웹소켓 연결을 위한 엔드포인트 등록
    - 클라이언트는 해당 엔드포인트를 통해 WebSocket 연결을 수립할 수 있다.


FRONTEND
---
클라이언트 측은 서버 측에 메시지를 보내고 서버 측으로부터 메시지를 받는다.

### index.html
``` html
<!DOCTYPE html>
<html>
<head>
    <title>Hello WebSocket</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    <link href="/main.css" rel="stylesheet">
    <script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@stomp/stompjs@7.0.0/bundles/stomp.umd.min.js"></script>
    <script src="/app.js"></script>
</head>
<body>
<noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
    enabled. Please enable
    Javascript and reload this page!</h2></noscript>

...
```
- 웹소켓을 통해 스톰프를 통해 서버와 통신하는 데 사용되는 StompJS 자바스크립트 라이브러리를 가져옴
- 클라이언트 애플리케이션의 로직이 포함된 app.js도 가져옴
- 나머지는... 뭐 웹소켓 연결할거냐 말거냐 버튼이랑 이름 입력하는 버튼이랑 리스트... 같은거


### app.js
``` js
const stompClient = new StompJs.Client({
    brokerURL: 'ws://localhost:8080/gs-guide-websocket'
});

stompClient.onConnect = (frame) => {
    setConnected(true);
    console.log('Connected: ' + frame);
    stompClient.subscribe('/topic/greetings', (greeting) => {
        showGreeting(JSON.parse(greeting.body).content);
    });
};

... 중략 ...

function setConnected(connected) {
    $("#connect").prop("disabled", connected);
    $("#disconnect").prop("disabled", !connected);
    if (connected) {
        $("#conversation").show();
    }
    else {
        $("#conversation").hide();
    }
    $("#greetings").html("");
}

function connect() {
    stompClient.activate();
}

function disconnect() {
    stompClient.deactivate();
    setConnected(false);
    console.log("Disconnected");
}

function sendName() {
    stompClient.publish({
        destination: "/app/hello",
        body: JSON.stringify({'name': $("#name").val()})
    });
}

function showGreeting(message) {
    $("#greetings").append("<tr><td>" + message + "</td></tr>");
}

$(function () {
    $("form").on('submit', (e) => e.preventDefault());
    $( "#connect" ).click(() => connect());
    $( "#disconnect" ).click(() => disconnect());
    $( "#send" ).click(() => sendName());
});
```
- 처음에 생성한 stompClient 객체는 웹소켓 서버가 연결을 기다리는 경로인 /gs-guide-websocket을 참조하는 brokerURL로 초기화됨. 'ws://localhost:8080/gs-guide-websocket' 이렇게 설정해줬는데 이 주소는 WebSocketConfig에서 설정했던 엔드포인트임<br/>
``` java
@Override
public void registerStompEndpoints(StompEndpointRegistry registry) {
registry.addEndpoint("/gs-guide-websocket");
}
```


- `stompClient.onConnect`
    - `(frame) => { ... }`: stompClient가 연결될 때 실행할 함수를 정의
    - `setConnected(true)`: 연결 상태를 나타내는 함수인 setConnected를 호출헤 연결 상태를 true로 설정<br/>
        -> 연결이 성공적으로 이루어짐
    - `stompClient.subscribe('/topic/greetings', (greeting) => {showGreeting(...)})`<br/>
        : /topic/greetings라는 특정 주제 구독(subscribe)
        - 서버가 해당 주제에 새로운 메시지를 발행할 때마다 클라이언트가 이를 받게 됨
        - 새로운 메시지가 도착하면 해당 메시지를 처리하기 위해 showGreeting 함수를 호출
    - `showGreeting(JSON.parse(greeting.body).content)`<br/>
        : 받은 메시지의 내용을 화면에 표시

- `sendName`
    <br/>: 사용자가 입력한 이름을 JSON 형식으로 포장하여 서버의 "/app/hello" 엔드포인트로 전송
    - `stompClient.publish({ ... })`: stompClient 객체를 사용하여 메시지를 발행<br/>
    -> 서버에 메시지 전송 가능
    - `destination: "/app/hello"`: 전송할 메시지의 목적지 지정
        - "/app/hello"로 지정함(GreetingController.greeting()이 수신할 대상)<br/>
            -> 이 메시지는 서버의 "/app/hello" 엔드포인트로 전송됨
    - `body: JSON.stringify({'name': $("#name").val()})`<br/>
        : 전송할 메시지의 내용을 정의
        -  사용자가 입력한 이름을 포함하는 JSON 형식의 데이터를 만듦
        - $("#name").val()를 통해 HTML 요소 중 ID가 "name"인 입력 필드의 값을 가져와서 이를 메시지에 포함시킴
        - 이후 JSON.stringify() 함수를 사용하여 JavaScript 객체를 JSON 문자열로 변환


## 실제 구현할 때 고려할 점
- 팀마다 채팅방 있음, 채팅방 식별자는 여행멤버모집게시글번호
실제 구현했던 코드 참고:<br/>
``` java
public void saveChat(ChatMessageRequest req) {
    ...
    template.convertAndSend("/topic/chat/" + req.getId(), req);
    ...
}
```
ChatMessageRequest는 채팅방id/보내는사람id/msg로 이루어진 DTO


- 채팅만 따로 떼어서 noSQL에서 관리하기로 했던 것 같은데 db도 함 조사해보기
- 협업 어떻게 구현할지 조사하기







참고자료
---
- [sanitize-html 모듈 사용법](https://inpa.tistory.com/entry/NODE-%EB%B3%B4%EC%95%88-%F0%9F%93%9A-sanitize-html-%EB%AA%A8%EB%93%88-%EC%82%AC%EC%9A%A9%EB%B2%95)
- [[컴퓨터 용어] echo 에코, 반향, 잔향](https://flowerkelly.tistory.com/entry/%EC%BB%B4%ED%93%A8%ED%84%B0-%EC%9A%A9%EC%96%B4-echo-%EC%97%90%EC%BD%94-%EB%B0%98%ED%96%A5-%EC%9E%94%ED%96%A5)