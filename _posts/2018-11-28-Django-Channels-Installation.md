---
layout: post
title: Django Channels - 설치하기 (installation)
---

# channels 설치하기
### 우선 설치 조건
- python 3.x version
- django 2.x version
- New Django Project(myproject) & APP(myapp)

#### 1. pip를 이용해 channels 라이브러리 설치
```
pip install -U channels
```

#### 2. django project 의 settings.py 설정
##### Installed Apps 에 channels 추가하기
``` python
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    ...
    'channels',
)
```

#### 3. `myproject/routing.py` 파일 생성 및 수정 - 기본 라우팅에 관한 코드
``` python
from channels.routing import ProtocolTypeRouter

application = ProtocolTypeRouter({
    # Empty for now (http->django views is added by default)
})
```

#### 4. `ASGI_APPLICATION` 에 관한 세팅을 settings.py 에 추가
``` python
ASGI_APPLICATION = "myproject.routing.application"
```
3번에서 만든 routing.py 파일을 라우팅 객체로 쓰겠다는 설정
