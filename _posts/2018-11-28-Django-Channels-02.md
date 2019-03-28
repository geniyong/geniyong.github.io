---
layout: post
title: Django Channels - 02. 채팅 서버 설정 (Implement Chat Server)
---
## 목표 : 양자간 커뮤니케이션 가능한 채팅방 개설
### 1. `chat/templates/chat/room.html` 채팅방 view 템플릿 만들기
```
chat/
    __init__.py
    templates/
        chat/
            index.html
            room.html
    urls.py
    views.py
```
#### `chat/templates/chat/room.html` 파일에 아래의 템플릿 코드 입력

``` html
<!-- chat/templates/chat/room.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Room</title>
</head>
<body>
    <textarea id="chat-log" cols="100" rows="20"></textarea><br/>
    <input id="chat-message-input" type="text" size="100"/><br/>
    <input id="chat-message-submit" type="button" value="Send"/>
</body>
<script>
    var roomName = {{ room_name_json }};

    var chatSocket = new WebSocket(
        'ws://' + window.location.host +
        '/ws/chat/' + roomName + '/');

    chatSocket.onmessage = function(e) {
        var data = JSON.parse(e.data);
        var message = data['message'];
        document.querySelector('#chat-log').value += (message + '\n');
    };

    chatSocket.onclose = function(e) {
        console.error('Chat socket closed unexpectedly');
    };

    document.querySelector('#chat-message-input').focus();
    document.querySelector('#chat-message-input').onkeyup = function(e) {
        if (e.keyCode === 13) {  // enter, return
            document.querySelector('#chat-message-submit').click();
        }
    };

    document.querySelector('#chat-message-submit').onclick = function(e) {
        var messageInputDom = document.querySelector('#chat-message-input');
        var message = messageInputDom.value;
        chatSocket.send(JSON.stringify({
            'message': message
        }));

        messageInputDom.value = '';
    };
</script>
</html>
```
### 2. chat/views.py 파일에 room.html 뷰 함수 만들기

``` python
# chat/views.py
from django.shortcuts import render
from django.utils.safestring import mark_safe
import json

def index(request):
    return render(request, 'chat/index.html', {})

def room(request, room_name):
    return render(request, 'chat/room.html', {
        'room_name_json': mark_safe(json.dumps(room_name))
    })
```

### 3. chat/urls.py 파일에 URL 설정 코드 입력
# chat/urls.py
``` python
# chat/urls.py
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^(?P<room_name>[^/]+)/$', views.room, name='room'),
]
```
### 4. django 서버 실행
`$ python3 manage.py runserver`

### 5. 채팅방 개설
`http://127.0.0.1:8000/chat/` 경로로 이동 후 방이름 란에 `lobby` 입력

##### 채팅방에 접속 시 room 뷰에 의해서 웹소켓에 접속을 시도하지만 아직 컨슈머를 만들지 않음
웹소켓에 접속을 시도하는 코드는 room.html 의 자바스크립트 코드 임.          
웹소켓 URL = ws://127.0.0.1:8000/ws/chat/lobby/
![image](https://user-images.githubusercontent.com/35215439/49091990-59cd8680-f2a4-11e8-83e2-a01085667290.png)

### 6. 첫번째 컨슈머 만들기
채널, 웹소켓, 컨슈머의 상호작용은 django 의 http request 및 response와 비슷하다      
django 의 http 통신은 다음과 같이 이루어짐          
1) django 에서 http url 을 통해 http request 가 발생함            
2) 그러면 django 의 root URL 부터 해당 URL 에 전달된 view 함수를 찾음     
3) 마지막으로 해당 view 함수를 실행하고 response 를 리턴     

이와 유사하게 채널, 웹소켓, 컨슈머의 상호작용은 다음과 같이 이루어짐       
1) channel 이 websocket 의 연결을 허용 (위의 1번)     
2) 루트 라우팅 설정부터 컨슈머를 찾음 (위의 2번)    
3) 마지막으로 해당 컨슈머의 다양한 기능을 호출하여 연결된 이벤트를 처리 (위의 3번)
----------------       
`/ws/chat/ROOM_NAME/` 경로로 WebSocket 을 연결하는 컨슈머를 만듦     
#### `chat/consumers.py` 파일 생성 및 코드 작성   
```
chat/
    __init__.py
    consumers.py
    templates/
        chat/
            index.html
            room.html
    urls.py
    views.py
```

``` python
# chat/consumers.py
from channels.generic.websocket import WebsocketConsumer
import json

class ChatConsumer(WebsocketConsumer):
    def connect(self):
        self.accept()

    def disconnect(self, close_code):
        pass

    def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        self.send(text_data=json.dumps({
            'message': message
        }))
```
##### NOTE!
This is a synchronous WebSocket consumer that accepts all connections, receives messages from its client, and echos those messages back to the same client. For now it does not broadcast messages to other clients in the same room.

Channels also supports writing asynchronous consumers for greater performance. However any asynchronous consumer must be careful to avoid directly performing blocking operations, such as accessing a Django model. See the Consumers reference for more information about writing asynchronous consumers.

[ 현재 코드는 동기식 컨슈머인데 이렇게 되버리면 같은 채팅방에 있는 다른 사용자들한테 내가 보낸 메시지를 브로드캐스팅 하지 않음 물론 비동기식 컨슈머가 더 나은 성능을 지원하지만, 비동기식 컨슈머를 만들 때에, django의 모델에 대한 접근과 같은 blocking operation(?)을 직접 수행하지 않도록 신중하게 개발해야함]     
여기까지가 공식 문서 내용이고, 사족 붙이면 위의 이유때문에 동기식으로 디폴트로 해놓은게 아닐까 싶음

### 7. `chat/routing.py` 파일 생성 및 코드 작성
```
chat/
    __init__.py
    consumers.py
    routing.py
    templates/
        chat/
            index.html
            room.html
    urls.py
    views.py
```

``` python
# chat/routing.py
from django.conf.urls import url

from . import consumers

websocket_urlpatterns = [
    url(r'^ws/chat/(?P<room_name>[^/]+)/$', consumers.ChatConsumer),
]
```

### 8. Websocket Routing URL 의 root URL Conf 작업
앞선 Django 의 HTTP 통신 과정에서 알아봤듯이, websocket 의 통신 과정에서도    
root routing URL 로 부터 컨슈머를 찾기 시작하기 때문에        
root routing URL Conf 작업을 진행함
`mysite/routing.py` 경로의 root routing.py 코드 작성      
``` python
# mysite/routing.py
from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
import chat.routing

application = ProtocolTypeRouter({
    # (http->django views is added by default)
    'websocket': AuthMiddlewareStack(
        URLRouter(
            chat.routing.websocket_urlpatterns
        )
    ),
})
```

### 9. DB migrate 후 runserver
```
$ python manage.py makemigrations
$ python manage.py migrate
$ python manage.py runserver
```

### 10. 채팅방에서 메시지 전송해보기
`http://127.0.0.1:8000/chat/lobby/` 경로로 이동 후 메시지 입력    
입력한 메시지가 그대로 echo 되어 출력 됨    
하지만 새로운 창을 열어서 같은 곳에 접속 시 해당 내용은 볼 수 없음   
진정한 채팅이 가능하려면 조금 전 만든 컨슈머를 채팅인원수만큼 만들어야 함    
#### Channel Layer : 채널 레이어가 여러 사람이 채팅을 할 수 있게 지원함
#### Channel Layer 설정은 다음 포스팅에서 진행...
