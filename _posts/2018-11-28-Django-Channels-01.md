---
tags: Django
title: Django Channels - 01. 기본설정
---

###### python3.x 및 django 2.x 의 설치가 되어있다고 간주.
### 1. mysite 프로젝트 생성 및 chat 어플리케이션 생성
```
$ django-admin startproject mysite
$ python3 manage.py startapp chat
```

두 명령어를 실행하고 나면 다음과 같은 디렉터리 구조가 생성된다
```
- mysite/
--- manage.py
--- mysite/
------  __init__.py
------  settings.py
------  urls.py
------  wsgi.py
--- chat/
------ __init__.py
------ admin.py
------ apps.py
------ migrations/
--------- __init__.py
------ models.py
------ tests.py
------ views.py
```

### 2. mysite/settings.py 세팅 파일 설정
``` python
# mysite/settings.py
INSTALLED_APPS = [
    'chat',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

### 3. index view 만들기 (채팅을 위한 웹페이지 뷰)
- chat/ 하위 경로에 templates 디렉토리 및 파일 생성
```
chat/
    __init__.py
    ...

    templates/
        chat/
            index.html
    
    ...
    views.py
```

### 4. `chat/templates/chat/index.html` 파일에 아래의 html 템플릿 코드 입력
``` html
<!-- chat/templates/chat/index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Rooms</title>
</head>
<body>
    What chat room would you like to enter?<br/>
    <input id="room-name-input" type="text" size="100"/><br/>
    <input id="room-name-submit" type="button" value="Enter"/>

    <script>
        document.querySelector('#room-name-input').focus();
        document.querySelector('#room-name-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#room-name-submit').click();
            }
        };

        document.querySelector('#room-name-submit').onclick = function(e) {
            var roomName = document.querySelector('#room-name-input').value;
            window.location.pathname = '/chat/' + roomName + '/';
        };
    </script>
</body>
</html>
```

### 5. `chat/views.py` 파일에 아래의 index 템플릿 렌더링 뷰 코드 입력
``` python
# chat/views.py
from django.shortcuts import render

def index(request):
    return render(request, 'chat/index.html', {})
```

### 6. `chat/urls.py` 파일에 아래의 URL 설정 코드 입력
``` python
# chat/urls.py
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```

### 7. `mysite/urls.py`파일에 아래의 URL 설정 코드 입력
(7번의 URL 경로 설정이 root 경로와 관련된 상위개념의 경로 설정임)

(6번의 URL 경로 설정이 7번에서 설정된 경로의 하위경로에 관한 경로 설정임)
``` python
# mysite/urls.py
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^chat/', include('chat.urls')),
    url(r'^admin/', admin.site.urls),
]
```

### 8. `python manage.py runserver` 로 서버 실행
`http://127.0.0.1:8000/chat/` 경로로 이동 후 조금 전 만든 index.html 파일 렌더링 확인
