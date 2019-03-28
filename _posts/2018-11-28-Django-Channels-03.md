---
tags: Django
title: Django Channles - 03. 채널 레이어 활성화 (Enable a Channel Layer)
---
## 채널레이어
### 채널레이어의 역할
> 여러 개의 컨슈머들이 서로 간에 커뮤니케이션을 하게 함       
> 여러 개의 컨슈머들이 장고의 다른 부분들과 커뮤니케이션을 하게 함    

### 채널레이어가 제공하는 두 개의 추상화
1. `Channel`
> 채널은 메시지의 목표 지점에 해당하는 메시지박스 역할을 함    
> 채널 이름만 알고 있으면 누구든 해당 채널에 메시지를 보낼 수 있음

2. `Group`
> 채널과 관련된 그룹임    
> 그룹에는 여러 개의 채널들을 추가 또는 삭제 할 수 있음     
> 그룹 이름만 알고 있으면 해당 그룹으로 메시지를 보내게 되는데 그러면 그룹 내의 모든 채널들에게 메시지가 전송 됨      
> 단, 특정 그룹에 어떤 채널들이 있는지를 열거할 순 없음

#### `NOTE`
```
모든 컨슈머는 자동으로 생성된 고유한 채널 이름을 가짐    
하나의 컨슈머가 메시지를 자신의 채널로 보내면 다른 컨슈머들은 해당 채널 이름을 모르기 때문에 메시지를 전달받을 수 없음     
그래서 필요한 것이 `그룹`이라는 추상화 개념임   
하나의 채팅룸에 접속한 컨슈머들끼리 채팅을 하도록 하려면  
같은 채팅룸에 있는 각각의 컨슈머들이 가지고 있는 고유한 채널 이름을    
하나의 그룹으로 묶어 주고 메시지를 보내는 목표 지점을 그룹으로 설정해 주면   
하나의 컨슈머가 보내는 메시지는 결국 다른 모든 컨슈머의 채널로 전송이 됨   
```

### 1. `Redis` : 채널레이어의 백업 저장소로 사용될 서버
Redis 서버 시작 (6379 포트로 열기)   
```
$ docker run -p 6379:6379 -d redis:2.8
```

### 2. `channels_redis` 설치 및 세팅
``` 
$ pip3 install channels_redis
```
``` python
# `mysite/settings.py` 설정 파일에 추가
# mysite/settings.py
# Channels
ASGI_APPLICATION = 'mysite.routing.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            "hosts": [('127.0.0.1', 6379)],
        },
    },
}
```

### 3. `chat/consumers.py` 파일 수정
``` python
# chat/consumers.py
from asgiref.sync import async_to_sync
from channels.generic.websocket import WebsocketConsumer
import json

class ChatConsumer(WebsocketConsumer):
    def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = 'chat_%s' % self.room_name

        # Join room group
        async_to_sync(self.channel_layer.group_add)(
            self.room_group_name,
            self.channel_name
        )

        self.accept()

    def disconnect(self, close_code):
        # Leave room group
        async_to_sync(self.channel_layer.group_discard)(
            self.room_group_name,
            self.channel_name
        )

    # Receive message from WebSocket
    def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        # Send message to room group
        async_to_sync(self.channel_layer.group_send)(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message
            }
        )

    # Receive message from room group
    def chat_message(self, event):
        message = event['message']

        # Send message to WebSocket
        self.send(text_data=json.dumps({
            'message': message
        }))
```

### 4. 채팅방 접속하여 테스트
```
$ redis-server & python manage.py runserver
```
