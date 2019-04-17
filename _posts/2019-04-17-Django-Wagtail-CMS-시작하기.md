---
tags: Django, Wagtail
title: Django-Wagtail-CMS-시작하기
---
## 들어가기전
wagtail 은 python + django 기반의 CMS 오픈소스이다. 대표적인 django 기반의 CMS 오픈소스로는 익히 알려진 django-cms 오픈소스가 있지만, wagtail 또한 django-cms 못지 않게 활발히 활동중인 오픈소스 프로젝트이다.
```
     wagtail              |     django-cms
------------------------------------------------
9,068 commits             | 15,799 commits
29 branches               | 15 branches
101 releases              | 121 releases
343 contributors          | 392 contributors
```

특히 wagtail 의 경우 더 나은 UI 요소와 저명한 참여 조직들(구글, 나사 등) 덕분에 기대가 되는 오픈소스 프로젝트다.

wagtail 을 시작해보는 과정은 두 방법으로 나뉜다.           
하나는 wagtail 프로젝트를 설치하여 시작하는 방법.
하나는 기존의 django 프로젝트 내에 wagtail app 을 추가하는 방법.

사실 wagtail 은 django app 형태로 되어있다. 그래서 기존의 django 프로젝트와 통합하기도 쉽다.

## wagtail 로 첫 프로젝트 생성하기
`note`
``` 
이 부분에서는 완전 새로운 wagtail 프로젝트를 설정하는 방법을 다루고 있다. 만약 기존에 존재하는 프로젝트와 wagtail 을 통합하고자 한다면 아래의 "django 프로젝트와 wagtail 의 통합" 을 참조.
```
### 설치 및 서버 실행
1. wagtail 과 종속성 설치
``` bash
pip install wagtail
```

2. 사이트 시작하기
``` bash
wagtail start mysite
cd mysite
```
wagtail 은 `start` 명령어를 제공하는데 이 명령어는 `django-admin.py startproject` 명령어와 비슷한 역할을 한다. `wagtail start mysite` 를 실행하면, 새로운 `mysite` 폴더를 생성한다. 이 폴더 내부에는 몇 가지 wagtail 만을 위해 필요한 프로젝트 설정을 포함해 빈 `HomePage` 모델과 기본 `template` 을 가지고 있는 "home" 앱과 샘플 "search" 앱을 포함하고 있다.

3. 프로젝트 종속성 설치하기
``` bash
pip install -r requirements.txt
```

4. 데이터베이스 생성하기
``` bash
python manage.py migrate
```

5. 관리자 유저 생성하기
```bash
python manage.py createsuperuse
```

6. 서버 시작하기
```bash
python manage.py runserver
```
서버 시작 후 http://127.0.0.1:8000 접속하면 wagtail 웰컴페이지를 볼 수 있다.     
http://127.0.0.1:8000/admin : 어드민 페이지로 이동

### HomePage 모델 확장하기
"home" 앱에서 `models.py` 내부의 비어있는 `HomePage` 모델을 정의하고 마이그레이션 하는 즉시 홈페이지를 만들고 wagtail 이 이를 사용할 수 있도록 구성한다.

`body` 필드를 추가하기 위해 `home/models.py` 를 다음과 같이 수정:
```python
from django.db import models

from wagtail.core.models import Page
from wagtail.core.fields import RichTextField
from wagtail.admin.edit_handlers import FieldPanel


class HomePage(Page):
    body = RichTextField(blank=True)

    content_panels = Page.content_panels + [
        FieldPanel('body', classname="full"),
    ]
```
`body` 필드는 특별한 Wagtail 필드에 해당하는 `RichTextField` 로 정의된다. 물론 어떤 [django core fields](https://docs.djangoproject.com/en/stable/ref/models/fields/) 를 사용해도 상관없다. `content_panels` 는 용량을 정의하고 편집 인터페이스의 레이아웃을 정의한다. [페이지 모델 만들기 더 알아보기](http://docs.wagtail.io/en/v2.4/topics/pages.html)

`python manage.py makemigrations` 와 `python manage.py migrate` 를 실행하면 방금 바뀐 모델이 데이터베이스에 적용된다. 이렇게 두개의 명령어는 모델에 대한 정의를 변경할 때 마다 그 즉시 실행해야지만 변경 내용을 데이터베이스 적용할 수 있다.

이제 wagtail admin 영역에서 새로운 `body` 필드에 대해 홈페이지를 수정할 수 있다. `body` 필드에 아무 text 를 입력하고 페이지를 발행해보자.

이제 모델 변경사항을 반영하기 위해서 페이지 템플릿을 업데이트 해야한다. wagtail 은 각 페이지를 렌더하기 위해 일반적인 django templates 을 사용한다. 기본적으로, 앱과 모델 이름에서 형성된 템플릿 파일 이름을 찾을 것이다. 이는 대문자가 밑줄로 구분되는 형식이다. (예를 들어 "home" 앱 내의 `HomePage` 는 `home/home_page.html` 이 됨) 이 템플릿 파일은 [Django 의 템플릿 규칙](https://docs.djangoproject.com/en/stable/intro/tutorial03/#write-views-that-actually-do-something) 에 의한 위치만 맞다면 어디서든 인식될 수 있다; 편의상으로 앱내부의 `templates` 폴더 내에 위치한다.

`home/templates/home/home_page.html` 파일을 다음과 같이 수정한다:
```html
{% raw %}
{% extends "base.html" %}

{% load wagtailcore_tags %}

{% block body_class %}template-homepage{% endblock %}

{% block content %}
    {{ page.body|richtext }}
{% endblock %}
{% endraw %}
```

#### wagtail 템플릿 태그
wagtail 은 수많은 [템플릿 태그와 필터](http://docs.wagtail.io/en/v2.4/topics/writing_templates.html#template-tags-and-filters)를 제공한다. 그리고 이들은 `{% raw %}{% load wagtailcore_tags %}{% endraw %}` 를 템플릿 파일의 최상단에 위치시킴으로써 불러올 수 있다.

이 튜토리얼에서는 `RichTextField` 의 내용을 escape 및 출력하기 위해 `richtext` filter를 사용한다:
```html
{% raw %}
{% load wagtailcore_tags %}
{{ page.body|richtext }}
{% endraw %}
```
그리고 이는 다음의 코드를 만들어낸다:
``` html
<div class="rich-text">
    <p>
        <b>Welcome</b> to our new site!
    </p>
</div>
```

`note` : `{% raw %}{% load wagtailcore_tags %}{% endraw %}` 을 wagtail 의 태그를 사용하는 템플릿의 최상단에 입력해야한다. 그렇지 않으면 django 가 `TemplateSyntaxError` 를 통해 예외처리 할 것이다.

### 기본적인 블로그
이제 블로그를 만들어 볼 것이다. 블로그를 만들기 위해서는 `python manage.py startapp blog` 를 실행하여 `blog` 라는 새로운 앱을 생성해야 한다.

새로운 `blog` 앱을 생성하고 나면 `mysite/settings/base.py` 세팅 파일에서 `INSTALLED_APPS`
부분에 `blog` 를 추가해줘야 한다.

#### 블로그 목차 및 게시물
블로그의 간단한 목차 페이지를 위해 `blog/models.py` 를 다음과 같이 작성하자:
```python
from wagtail.core.models import Page
from wagtail.core.fields import RichTextField
from wagtail.admin.edit_handlers import FieldPanel


class BlogIndexPage(Page):
    intro = RichTextField(blank=True)

    content_panels = Page.content_panels + [
        FieldPanel('intro', classname="full")
    ]
```

`python manage.py makemigrations` 와 `python manage.py migrate`  를 실행한다.

페이지 모델의 이름이 `BlogIndexPage` 라고 정의되었기 때문에, 기본 템플릿 이름은(오버라이딩 하지 않는다면) `blog/templates/blog/blog_index_page.html` 이 될 것이다. 이 파일은 다음의 내용으로 작성하자:
```html
{% raw %}
{% extends "base.html" %}

{% load wagtailcore_tags %}

{% block body_class %}template-blogindexpage{% endblock %}

{% block content %}
    <h1>{{ page.title }}</h1>

    <div class="intro">{{ page.intro|richtext }}</div>

    {% for post in page.get_children %}
        <h2><a href="{% pageurl post %}">{{ post.title }}</a></h2>
        {{ post.specific.intro }}
        {{ post.specific.body|richtext }}
    {% endfor %}

{% endblock %}
{% endraw %}
```
대부분은 익숙할 것이지만, `get_children` 에 대해서는 나중에 설명할 것이다. Django 의 `url` 태그와 비슷하지만 wagtail 페이지 객체만을 인자로 사용하는 `pageurl` 태그를 주의하자.

wagtail admin 페이지에서 `HomePage` 의 자식으로 `BlogIndexPage` 를 생성하자. 그리고 Promote 탭에서 "blog" 를 슬러그로 설정한 뒤 발행한다. 이제 `/blog` 경로로 접근할 수 있게 되었다. 

이제 blog 게시글에 대한 모델과 템플릿이 필요하다. `blog/models.py` 작성 :
```python
from django.db import models

from wagtail.core.models import Page
from wagtail.core.fields import RichTextField
from wagtail.admin.edit_handlers import FieldPanel
from wagtail.search import index


class BlogIndexPage(Page):
    intro = RichTextField(blank=True)

    content_panels = Page.content_panels + [
        FieldPanel('intro', classname="full")
    ]


class BlogPage(Page):
    date = models.DateField("Post date")
    intro = models.CharField(max_length=250)
    body = RichTextField(blank=True)

    search_fields = Page.search_fields + [
        index.SearchField('intro'),
        index.SearchField('body'),
    ]

    content_panels = Page.content_panels + [
        FieldPanel('date'),
        FieldPanel('intro'),
        FieldPanel('body', classname="full"),
    ]
```

`python manage.py makemigrations` 와 `python manage.py migrate` 를 실행한다.

`blog/templates/blog/blog_page.html` 를 다음과 같이 작성한다:
``` html
{% raw %}
{% extends "base.html" %}

{% load wagtailcore_tags %}

{% block body_class %}template-blogpage{% endblock %}

{% block content %}
    <h1>{{ page.title }}</h1>
    <p class="meta">{{ page.date }}</p>

    <div class="intro">{{ page.intro }}</div>

    {{ page.body|richtext }}

    <p><a href="{{ page.get_parent.url }}">Return to blog</a></p>

{% endblock %}
{% endraw %}
```
이 게시물이 속한 블로그의 URL 을 얻기 위해서 Wagtail 의 내장 `get_parent()` 메서드를 사용한다는 점을 유의하자.

이제 `BlogIndexPage` 의 자식으로써 몇 개의 블로그 게시물을 생성하자. 게시물을 만들때에는 "Blog Page" 유형을 확실히 선택해야 한다.

wagtail 은 다양한 부모 컨텐츠 유형중에서 무슨 종류의 컨텐츠가 만들어질 수 있는지 완전히 제어할 수 있게 해준다. 기본적으로는 모든 페이지 유형은 또 다른 페이지 유형의 자식이 될 수 있다.   

이제 블로그가 작동하기 시작할 것이다. `/blog` 경로로 접근하면 작성한 게시물을 볼 수 있다.

`Titles` 은 게시물 페이지로의 링크가 되야하며, 다시 블로그의 홈페이지로의 링크도 게시물 페이지의 `footer` 영역에서 나타나야 한다.

#### 부모와 자식
Watail 에서 하게 될 대부분의 작업들은 계층적인 "트리" 구조의 개념에 포함된다. 이 계층 구조는 `nodes` 와 `leaves` 로 구성되어 있다. [트리 이론 참조](http://docs.wagtail.io/en/v2.4/reference/pages/theory.html). 이 경우에 `BlogIndexPage` 는 하나의 "node" 이며, 각각의 `BlogPage` 인스턴스들이 "leaves" 에 해당한다.

```
               BlogIndexPage
                    |
         ----------------------
        |           |          |
    BlogPage     BlogPage   BlogPage
```

`blog_index_page.html` 의 내부를 한번더 보자:
``` html
{% raw %}
{% for post in page.get_children %}
    <h2><a href="{% pageurl post %}">{{ post.title }}</a></h2>
    {{ post.specific.intro }}
    {{ post.specific.body|richtext }}
{% endfor %}
{% endraw %}
```
wagtail 에서의 모든 "page" 는  그 페이지가 속해 있는 계층 지위로 부터 부모 또는 자식을 호출 할 수 있다. 그런데 왜 `post.intro` 대신에 `post.specific.intro` 를 특정해야할까? 이는 모델에서 지정한 방식과 관련이 있기 때문이다:

`class BlogPage(Page)`:

`get_children()` 메서드는 `Page` 라고 하는 베이스 모델의 인스턴스 목록을 가져오는 메서드다. 기본 클래스로부터 상속받은 인스턴스들의 특징들을 참조하기를 원할 때, wagtail 은 `specifc` 메서드를 제공한다. 따라서 여기서는 실제 `BlogPage` 레코드를 가져오기 위해 `specific` 메서드가 사용된다. "title" field 가 기본 `Page` 모델에 존재하는 반면에, "intro" field 는 `BlogPage` 모델에만 존재한다는 점에서 `intro` field 에 접근하기 위해서는 `.specific` 을 사용해서 접근해야한다.

Django 의 `with` 태그를 사용하여 위의 코드를 아래와 같이 작성할 수도 있다:
```html
{% raw %}
{% for post in page.get_children %}
    {% with post=post.specific %}
        <h2><a href="{% pageurl post %}">{{ post.title }}</a></h2>
        <p>{{ post.intro }}</p>
        {{ post.body|richtext }}
    {% endwith %}
{% endfor %}
{% endraw %}
```

Wagtail 코드를 더욱 커스터마이징 하게 될 때, 전체적인 `QuerySet` 들이 계층 탐색에 도움이 될 것이다:
``` python
# 'somepage' 라는 객체가 주어질 때:
MyModel.objects.descendant_of(somepage)
child_of(page) / not_child_of(somepage)
ancestor_of(somepage) / not_ancestor_of(somepage)
parent_of(somepage) / not_parent_of(somepage)
sibling_of(somepage) / not_sibling_of(somepage)
# ... and ...
somepage.get_children()
somepage.get_ancestors()
somepage.get_descendants()
somepage.get_siblings()
```
[페이지 쿼리셋 참조 더 알아보기](http://docs.wagtail.io/en/v2.4/reference/pages/queryset_reference.html)

### Context 오버라이딩
지금까지의 blog index view 에서 두 가지 문제점이 있다:
1. 일반적으로 블로그는 시간 역순으로 그 내용을 표시한다.
2. 발행된 내용만을 보여주고 있는지를 확인하고 싶다.
이 두 가지를 달성하기 위해서는 단지 템플릿에서 index page 의 자식을 가져오는 것보다 그 이상의 무언가를 할 필요가 있다. 따라서 모델 정의부분에서 `QuerySet` 에 대해 수정하고자 할 것이다. wagtail 은 오버라이딩이 가능한 `get_context()` 메서드를 통해서 이를 가능하게 한다. `BlogIndexPage` 모델을 다음과 같이 수정하자:
```python
class BlogIndexPage(Page):
    intro = RichTextField(blank=True)

    def get_context(self, request):
        # Update context to include only published posts, ordered by reverse-chron
        context = super().get_context(request)
        blogpages = self.get_children().live().order_by('-first_published_at')
        context['blogpages'] = blogpages
        return context
```
이 코드에서 바뀐 것은, 기존의 context 를 가져와서 사용자 `QuerySet` 을 만들고 가져온 context 에 이를 추가한 뒤 수정된 context 를 view 에 돌려준 것이 전부다. 추가적으로 `blog_index_page.html` 파일도 조금 수정해줘야 한다. 아래의 부분만을 바꿔주자:
`{% raw %} {% for post in page.get_children %} to {% for post in blogpages %} {% endraw %}`

이제 게시글 중 하나를 발행취소 해보자. 그러면 blog index page 에서 해당 게시글이 보이지 않아야 할 것이다. 그리고 가장 최근 발행된 게시물이 첫번째로 보이도록 정렬된 상태로 남아있을 것이다.

### 이미지


