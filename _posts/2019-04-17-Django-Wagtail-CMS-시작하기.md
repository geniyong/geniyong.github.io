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
이 부분에서는 완전 새로운 wagtail 프로젝트를 설정하는 방법을 다루고 있다. 만약 기존에 존재하는 프로젝트와 wagtail 을 통합하고자 한다면 아래의 "기존 django 프로젝트와 wagtail 의 통합" 을 참조.
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

### 기본적인 블로그 만들기
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
게시물에 이미지 갤러리를 첨부할 수 있는 기능을 추가하자. `body` rich text field 안에 간단하게 이미지를 추가하는 것이 가능하지만, 데이터베이스 내의 새로운 전용의 객체 유형으로 갤러리 이미지를 설정할 수 있다는 이점이 있다. 데이터베이스에서 새로운 전용의 객체 유형으로 갤러리 이미지를 설정하게 될 경우 rich text field 내에서 특정 방식으로 이미지를 배치하는 것보다도 템플릿에 있는 이미지의 레이아웃과 스타일을 완전히 제어할 수 있다는 장점이 있다. 또한 이미지를 블로그 글과는 별도로 어디서든 사용할 수 있다(예를 들어 blog index page 의 썸네일 이미지로 보여줄 수 있다)

새로운 `BlogGalleryImage` 모델을 `models.py` 에 추가하기:
```python
from django.db import models

# New imports added for ParentalKey, Orderable, InlinePanel, ImageChooserPanel

from modelcluster.fields import ParentalKey

from wagtail.core.models import Page, Orderable
from wagtail.core.fields import RichTextField
from wagtail.admin.edit_handlers import FieldPanel, InlinePanel
from wagtail.images.edit_handlers import ImageChooserPanel
from wagtail.search import index


# 기존의 모델 여기에 유지시켜야 함!!


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
        InlinePanel('gallery_images', label="Gallery images"),
    ]


class BlogPageGalleryImage(Orderable):
    page = ParentalKey(BlogPage, on_delete=models.CASCADE, related_name='gallery_images')
    image = models.ForeignKey(
        'wagtailimages.Image', on_delete=models.CASCADE, related_name='+'
    )
    caption = models.CharField(blank=True, max_length=250)

    panels = [
        ImageChooserPanel('image'),
        FieldPanel('caption'),
    ]
```
`python manage.py makemigrations` 와 `python manage.py migrate` 를 실행.

여기에는 몇 가지 새로운 개념들이 등장한다. 한번에 하나씩 알아보자.

`Orderable` 로 부터 상속받은 모델에서는 갤러리의 이미지 순서를 추적하기 위해서 `sort_order` 필드를 추가한다.

`BlogPage` 를 가리키는 `ParentalKey` 필드는 갤러리 이미지가 첨부 될 특정한 페이지이다. `ParentalKey` 는 `ForeignKey` 와 유사하게 작동하기도 하지만, `BlogPageGalleryImage` 를 `BlogPage` 모델의 자식으로 정의하기도 한다. 그래서 페이지를 중간 저장하거나 수정기록의 추적하는 작업에 있어서 자식이 되는 모델을 부모가 되는 페이지의 기본 요소로 취급하게 된다.

`image` 는 wagtail 의 내장 이미지 모델의 `ForeignKey` 이다. 이 내장 이미지 모델에 이미지들이 저장된다. `image` 는 `ImageChooserPanel` 이라는 전용 panel 유형과 함께 정의되는데, 이 판넬이 기존의 이미지 중 선택하거나 새로운 이미지를 업로드 할 수 있게 하는 팝업 인터페이스를 제공하게 된다. 이러한 방식은 한 이미지가 여러 갤러리에 존재하는 것을 가능케 한다. 한 이미지가 여러 갤러리에 존재할 수 있다는 점은 페이지와 이미지들 사이에서 다대다 관계를 만들어내는데 효과적이다.

외래키에 적용된 `on_delete=models.CASCADE` 옵션은 만약 이미지가 시스템에서 삭제되었을 때, 갤러리의 목록에서도 그 이미지를 지울 것인지에 대한 내용이다. 어떤 상황에서는 갤러리 목록은 그대로 남겨두는 것이 적절할지도 모른다. 예를 들어 "직원" 페이지에 얼굴 사진이 있는 사람들의 목록이 포함되어 있고, 그 사진들 중 하나가 삭제되었다 할 지라도, 그 사람의 사진 정보만 없이 그 페이지에서 그대로 두는 것이 더 나을 수 있다. 이러한 경우, 외래키 설정을 다음과 같이 할 수 있다: `blank=True, null=True, on_delete=models.SET_NULL`

마지막으로 `InlinePanel` 을 `BlogPage.content_panels` 에 추가하면 BlogPage 의 편집 인터페이스에서 갤러리 이미지를 사용할 수 있게 된다.

이미지를 포함시키기 위해 `blog_page.html` 를 수정한다:
```html
{% raw %}
{% extends "base.html" %}

{% load wagtailcore_tags wagtailimages_tags %}

{% block body_class %}template-blogpage{% endblock %}

{% block content %}
    <h1>{{ page.title }}</h1>
    <p class="meta">{{ page.date }}</p>

    <div class="intro">{{ page.intro }}</div>

    {{ page.body|richtext }}

    {% for item in page.gallery_images.all %}
        <div style="float: left; margin: 10px">
            {% image item.image fill-320x240 %}
            <p>{{ item.caption }}</p>
        </div>
    {% endfor %}

    <p><a href="{{ page.get_parent.url }}">Return to blog</a></p>

{% endblock %}
{% endraw %}
```
여기서 `<img>` element 를 넣기 위해서 `{% raw %}{% image %}{% endraw %}` 태그를 사용하고 있다. 이때 이 태그는 `wagtailimages_tags` 에 있기 때문에 템플릿 상단 부분에 이를 import 해야한다. `fill-320x240` 은 이미지의 크기가 조정되고 잘려서 320x240 의 사각형을 채우도록 하는 인자이다. [템플릿 안에서 이미지를 사용하는 법 더 알아보기](http://docs.wagtail.io/en/v2.4/topics/images.html)

갤러리 이미지는 이제 그 자체로 데이터베이스의 객체가 되었기 때문에, 블로그 게시물의 body 와는 별도로 독립적으로 이미지들을 조회하고 다시 사용할 수 있다.    
`main_image` 메서드를 정의해서 첫번째 갤러리 아이템으로부터 이미지를 리턴해보자(갤러리에 아이템이 존재하지 않는 다면 None 을 리턴):
```python
class BlogPage(Page):
    date = models.DateField("Post date")
    intro = models.CharField(max_length=250)
    body = RichTextField(blank=True)

    def main_image(self):
        gallery_item = self.gallery_images.first()
        if gallery_item:
            return gallery_item.image
        else:
            return None

    search_fields = Page.search_fields + [
        index.SearchField('intro'),
        index.SearchField('body'),
    ]

    content_panels = Page.content_panels + [
        FieldPanel('date'),
        FieldPanel('intro'),
        FieldPanel('body', classname="full"),
        InlinePanel('gallery_images', label="Gallery images"),
    ]
```
이 메서드는 이제 템플릿에서 사용할 수 있다. `blog_index_page.html` 템플릿을 업데이트하여 방금 만든 `main_image` 메서드를 포함시켜 게시물의 이미지를 썸네일 이미지로 따로 빼서 사용해보자:

```html
{% raw %}
{% load wagtailcore_tags wagtailimages_tags %}

...

{% for post in blogpages %}
    {% with post=post.specific %}
        <h2><a href="{% pageurl post %}">{{ post.title }}</a></h2>

        {% with post.main_image as main_image %}
            {% if main_image %}{% image main_image fill-160x100 %}{% endif %}
        {% endwith %}

        <p>{{ post.intro }}</p>
        {{ post.body|richtext }}
    {% endwith %}
{% endfor %}
{% endraw %}
```

### 게시물 태그 기능

편집자들이 게시물에 "태그" 를 부여하고 싶다고 해보자. 예를 들면 독자들이 자전거와 관련된 모든 컨텐츠만 모아서 볼 수 있도록 `자전거` 라는 태그를 달 수 있다. 이를 위해서는 wagtail 의 태그 시스템을 불러와 `BlogPage` 모델과 `content panel` 에 붙이고 `blog_post` 템플릿에 링크된 태그를 렌더링하기만 하면 된다. 물론, 태그별로 URL 뷰도 필요할 것이다.    

먼저, `models.py` 파일을 한번 더 수정하자:
``` python
from django.db import models

# New imports added for ClusterTaggableManager, TaggedItemBase, MultiFieldPanel

from modelcluster.fields import ParentalKey
from modelcluster.contrib.taggit import ClusterTaggableManager
from taggit.models import TaggedItemBase

from wagtail.core.models import Page, Orderable
from wagtail.core.fields import RichTextField
from wagtail.admin.edit_handlers import FieldPanel, InlinePanel, MultiFieldPanel
from wagtail.images.edit_handlers import ImageChooserPanel
from wagtail.search import index


# ... (Keep the definition of BlogIndexPage)


class BlogPageTag(TaggedItemBase):
    content_object = ParentalKey(
        'BlogPage',
        related_name='tagged_items',
        on_delete=models.CASCADE
    )


class BlogPage(Page):
    date = models.DateField("Post date")
    intro = models.CharField(max_length=250)
    body = RichTextField(blank=True)
    tags = ClusterTaggableManager(through=BlogPageTag, blank=True)

    # ... (Keep the main_image method and search_fields definition)

    content_panels = Page.content_panels + [
        MultiFieldPanel([
            FieldPanel('date'),
            FieldPanel('tags'),
        ], heading="Blog information"),
        FieldPanel('intro'),
        FieldPanel('body'),
        InlinePanel('gallery_images', label="Gallery images"),
    ]
```

`python manage.py makemigrations` 와 `python manage.py migrate` 를 실행

새롭게 `modelcluster`, `taggit` 을 import 해야함을 유의하자. 또한 새로운 `BlogPageTag` 모델을 추가하고 `BlogPage` 모델에는 `tags` 필드를 새로 추가해야함을 유의하자. `content_panels` 에서 `MultiFieldPanel` 을 사용하면 `date` 와 `tags` 필드를 그룹화하여 가독성을 높일 수 있다.

`BlogPage` 인스턴스들 중 하나를 수정하면 게시물에 태그를 지정할 수 있다.

`BlogPage` 에 태그를 렌더하기 위해서는 `blog_page.html` 템플릿에 아래를 추가하자:
``` html
{% raw %}
{% if page.tags.all.count %}
    <div class="tags">
        <h3>Tags</h3>
        {% for tag in page.tags.all %}
            <a href="{% slugurl 'tags' %}?tag={{ tag }}"><button type="button">{{ tag }}</button></a>
        {% endfor %}
    </div>
{% endif %}
{% endraw %}
```
여기서 주목할만한 점은 이전에 사용했던 `pageurl` 템플릿 태그 대신에 `slugurl` 태그를 사용해서 페이지를 연결하고 있다는 점이다. `slugurl` 과 `pageurl` 의 차이점은 `slugurl` 는 (Promote 탭에서 설정한) 페이지 슬러그를 인자로 받아들인다는 것이다. `pageurl` 이 명확하고 추가적인 데이터베이스 조회를 피한다는 점에서 더 많이 사용되지만, 지금처럼 반복문에서의 경우, 페이지 객체를 쉽게 구할 수 없기 때문에 덜 선호되는 슬러그 태그를 사용하기도 한다.

태그를 통해 게시물에 방문하게 되면 링크 버튼들이 (각 태그당 하나의 링크 버튼) 보일 것이다. 하지만 아직 "tags" view 를 정의하지 않았기 때문에 버튼을 클릭하면 404 가 나올 것이다. 다음을 `models.py` 에 추가하자:

```python
class BlogTagIndexPage(Page):

    def get_context(self, request):

        # Filter by tag
        tag = request.GET.get('tag')
        blogpages = BlogPage.objects.filter(tags__name=tag)

        # Update template context
        context = super().get_context(request)
        context['blogpages'] = blogpages
        return context
```

이 `Page` 를 기반으로 한 모델(=`BlogTagIndexPage`)은 자체 필드를 정의하지 않는 다는 점을 유의하자. 심지어 필드가 없다할지라도, `Page` 의 하위클래스는 자체 필드 없이도 wagtail 생태계의 일부로 만들기 때문에 admin 페이지에서 제목과 url 을 부여할 수 있고, `get_context()` 메서드에서 `QuerySet` 을 리턴하여 그 내용을 조작할 수 있다.

마이그레이션을 진행한 뒤, 새로운 `BlogTagIndexPage` 를 admin 페이지에서 진행한다. 아마 `BlogIndexPage` 와 병행하도록 `HomePage` 의 자식으로 새로운 페이지/뷰를 만들었을 것이다. Promote 탭에서 "tags" 라고 하는 슬러그를 부여하자.

`/tags` 으로 접근해보면 Django 에서 `blog/blog_tag_index_page.html` 템플릿을 만들어야 함을 알려 줄 것이다:
```html
{% raw %}
{% extends "base.html" %}
{% load wagtailcore_tags %}

{% block content %}

    {% if request.GET.tag|length %}
        <h4>Showing pages tagged "{{ request.GET.tag }}"</h4>
    {% endif %}

    {% for blogpage in blogpages %}

          <p>
              <strong><a href="{% pageurl blogpage %}">{{ blogpage.title }}</a></strong><br />
              <small>Revised: {{ blogpage.latest_revision_created_at }}</small><br />
              {% if blogpage.author %}
                <p>By {{ blogpage.author.profile }}</p>
              {% endif %}
          </p>

    {% empty %}
        No pages found with that tag.
    {% endfor %}

{% endblock %}
{% endraw %}
```

위의 내용 중 `Page` 모델에 내장된 `latest_revision_created_at` 필드를 호출하는 부분이 있다. 내장 된 필드기 때문에 페이지 인스턴스라면 언제든지 이를 이용할 수 있다는 점에서 편리하다.

아직 "author" 필드를 `BlogPage` 모델에 추가하지 않았으며, 아직 author 를 위한 `Profile` 모델도 추가하지 않았다. 이는 한번 알아서 추가해보도록 남겨두겠다.

BlogPost 아래의 태그버튼을 클릭하면 이제 페이지가 렌더 될 것이다.

### 카테고리
블로그에 카테고리 시스템을 추가해보자. 페이지 작성자가 단순히 페이지에서 태그를 사용함으로써 태그를 생성할 수 있었던 것과는 달리, 카테고리는 admin 페이지의 인터페이스의 별도 영역을 통해 사이트 소유자가 관리하는 고정된 목록이 될 것이다.

먼저 `BlogCategory` 를 정의하자. 카테고리는 그 자체로 페이지가 아니기 때문에 `Page` 를 상속받는것이 아니라 표준 Django `models.Model` 로 정의한다. Wagtail 은 관리 인터페이스를 통해 관리해야 하지만 페이지 트리 자체의 일부로 존재하지 않는 재사용 가능한 컨텐츠를 위해서 "snippets" 개념을 소개하고 있다.     
Django 의 표준 모델은 `@register_snippet` 데코레이터를 추가하여 `snippet` 으로 등록될 수 있다. 지금까지 페이지에서 사용한 모든 필드 유형은 `snippet` 에서도 사용할 수 있다. 여기서는 각 카테고리에 이름과 아이콘 이미지를 제공한다. `blog/models.py` 에 다음을 추가하자:
```python
from wagtail.snippets.models import register_snippet


@register_snippet
class BlogCategory(models.Model):
    name = models.CharField(max_length=255)
    icon = models.ForeignKey(
        'wagtailimages.Image', null=True, blank=True,
        on_delete=models.SET_NULL, related_name='+'
    )

    panels = [
        FieldPanel('name'),
        ImageChooserPanel('icon'),
    ]

    def __str__(self):
        return self.name

    class Meta:
        verbose_name_plural = 'blog categories'
```
`Note`
여기서 `content_panels` 대신에 `panels` 를 사용하고 있다는 점을 유의하자. `snippet` 은 일반적으로 `slug` 또는 `date` 와 같은 필드가 필요하지 않기 때문에 이들을 위한 편집 인터페이스는 일반적으로 제공되는 `content`/`promote`/`settings` 탭으로 분할되지 않으므로, 이에 맞게 `panel` 또한 `content_panel` 과 `promote_panel` 로 구별될 필요가 없다. 따라서 `content_panel` 이 아닌 `panel` 을 사용하고 있다.   

마이그레이션을 진행하고, admin 페이지의 메뉴에 나타나는 `Snippets` 영역을 통해 몇 개의 카테고리를 추가해보자.

이제 `BlogPage` 모델에 다대다 필드로 카테고리를 추가할 수 있다. 카테고리를 위해 사용할 필드 유형은 `ParentalManyToManyField` 이다. 이 필드는 Django 표준 `ManyToManyField` 의 변형이다. 앞서 `ParentalKey` 와 `ForeignKey` 의 차이와 같이, 페이지의 수정 기록에 대한 데이터를 저장하도록 보장한다.

``` python
# New imports added for forms and ParentalManyToManyField
from django import forms
from django.db import models

from modelcluster.fields import ParentalKey, ParentalManyToManyField
from modelcluster.contrib.taggit import ClusterTaggableManager
from taggit.models import TaggedItemBase

# ...

class BlogPage(Page):
    date = models.DateField("Post date")
    intro = models.CharField(max_length=250)
    body = RichTextField(blank=True)
    tags = ClusterTaggableManager(through=BlogPageTag, blank=True)
    categories = ParentalManyToManyField('blog.BlogCategory', blank=True)

    # ... (Keep the main_image method and search_fields definition)

    content_panels = Page.content_panels + [
        MultiFieldPanel([
            FieldPanel('date'),
            FieldPanel('tags'),
            FieldPanel('categories', widget=forms.CheckboxSelectMultiple),
        ], heading="Blog information"),
        FieldPanel('intro'),
        FieldPanel('body'),
        InlinePanel('gallery_images', label="Gallery images"),
    ]
```
`FieldPanel` 정의에서 `widget` 옵션 값을 보면 `기본 복수 선택 박스` 가 아니라 `체크 박스 기반` 위젯으로 설정하고 있는데, 이것이 좀 더 사용자 친화적으로 고려되기 때문이다.

마지막으로 카테고리를 표시하기 위해서 `blog_page.html` 템플릿을 변경한다:
```html
{% raw %}
<h1>{{ page.title }}</h1>
<p class="meta">{{ page.date }}</p>

{% with categories=page.categories.all %}
    {% if categories %}
        <h3>Posted in:</h3>
        <ul>
            {% for category in categories %}
                <li style="display: inline">
                    {% image category.icon fill-32x32 style="vertical-align: middle" %}
                    {{ category.name }}
                </li>
            {% endfor %}
        </ul>
    {% endif %}
{% endwith %}
{% endraw %}
```

## 기존 django 프로젝트와 wagtail 의 통합
wagtail 은 `wagtail start` 라는 명령어를 제공하고 wagtail 프로젝트를 가능한 빠르게 시작할 수 있도록 프로젝트 템플릿도 제공하고 있지만, 이미 존재하는 Django 프로젝트와으 통합도 쉽게 할 수 있다.

wagtail 은 현재 django 2.0 / 2.1 과 호환된다. 먼저 wagtail 패키지를 PyPI 로 부터 설치한다:
```bash
pip install wagtail
```

아니면 이미 존재하는 requirements 파일에 추가한다. 이렇게 설치하면 Pillow 라이브러리도 종속성에 의해 설치할 것이다.

### 설정
1) 기존 django 의 설정파일에서 `INSTALLED_APPS` 부분에 다음을 추가한다:

```python
[
    'wagtail.contrib.forms',
    'wagtail.contrib.redirects',
    'wagtail.embeds',
    'wagtail.sites',
    'wagtail.users',
    'wagtail.snippets',
    'wagtail.documents',
    'wagtail.images',
    'wagtail.search',
    'wagtail.admin',
    'wagtail.core',

    'modelcluster',
    'taggit',
]
```

2) `MIDDLEWARE` 부분에 다음을 추가한다:

```python
[
    'wagtail.core.middleware.SiteMiddleware',
    'wagtail.contrib.redirects.middleware.RedirectMiddleware',
]
```

3) `STATIC_ROOT` 세팅을 추가한다. (추가하지 않았을 경우에만)

``` python
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

4) `WAGTAIL_SITE_NAME` 을 추가한다. 이는 wagtail admin 백엔드의 메인 대시보드에서 보여진다.

```python
WAGTAIL_SITE_NAME = 'My Example Site'
```

이외에도 wagtail 을 위한 다양한 설정을 할 수 있다. [더 알아보기](http://docs.wagtail.io/en/v2.4/advanced_topics/settings.html)

### URL 설정
다음을 `urls.py` 파일에 추가한다:
```python
from django.urls import path, re_path, include

from wagtail.admin import urls as wagtailadmin_urls
from wagtail.documents import urls as wagtaildocs_urls
from wagtail.core import urls as wagtail_urls

urlpatterns = [
    #...
    re_path(r'^cms/', include(wagtailadmin_urls)),
    re_path(r'^documents/', include(wagtaildocs_urls)),
    re_path(r'^pages/', include(wagtail_urls)),
    #...
]
```

여기의 URL 경로들은 각자 프로젝트의 경로 스키마에 맞게 필요에 따라 바꿔도 된다.

`wagtailadmin_urls` 는 wagtail 을 위한 관리자 인터페이스를 제공한다. 이는 django 의 관리자 인터페이스(`django.contrib.admin`)와는 구분된다. wagtail 만 존재하는 프로젝트에서는 이 `wagtailadmin_urls` 가 `/admin/` 으로 설정 되어 있지만, 기존의 django 프로젝트와 통합하려는 경우에는 기존의 django admin 경로 역시 `/admin/` 이기 때문에, 충돌이 발생한다. 따라서 여기서의 `/cms/` 와 같이 대안의 경로를 사용할 수 있다.

`wagtaildocs_urls` 는 문서 파일이 제공되는 위치이다. wagtail 의 문서 관리 기능을 사용하지 않는 다면 생략해도 된다.

`wagtail_urls` 는 wagtail 사이트가 서비스되는 기본 위치이다. 위의 예시에서는 `/pages/` 경로 아래에서 wagtail 사이트가 서비스 되도록 되어 있지만, root URL 로서 서비스되길 원하면 다음과 같이 설정해도 된다:
``` python
re_path(r'', include(wagtail_urls)),
```

그런데 이 경우 `urlpatterns` 목록의 끝에 배치하여 더 구체적인 URL 패턴을 재정의 하지 않도록 해야한다.

마지막으로, `MEDIA_ROOT` 로부터 사용자가 업로드한 파일들을 서비스하기 위해서 설정해야한다. 이미 Django 프로젝트에서 관련 설정을 했더라면 진행할 필요가 없다:
```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... the rest of your URLconf goes here ...
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

하지만 이는 오직 개발모드(`DEBUG=True`) 에서만 작동한다. 배포모드에서는 웹서버 관련 설정을 통해 `MEDIA_ROOT` 로 부터 파일들을 서비스하도록 해야한다. 이는 Django 의 설정방법에 따르므로 더 자세한 내용은 [Serving files uploaded by a user during development](https://docs.djangoproject.com/en/stable/howto/static-files/#serving-files-uploaded-by-a-user-during-development) 와 [Deploying static files](https://docs.djangoproject.com/en/stable/howto/static-files/deployment/) 를 참조

지금까지의 설정대로라면 `python manage.py migrate` 를 통해 데이터베이스 테이블을 생성할 준비가 된 것이다.

### 사용자 계정
슈퍼유저의 계정은 자동으로 wagtail 관리자 인터페이스에 접근 권한을 가진다. `python manage.py createsuperuser` 를 통해 슈퍼유저를 생성할 수 있다. 몇 가지 제한사항을 가진 커스텀 유저 모델도 지원이 된다. wagtail 은 django 의 권한 프레임워크의 확장을 사용한다. 그래서 커스텀 유저 모델은 최소한 `AbstractBaseUser` 와 `PermissionMixin` 으로 부터 상속받아야 한다.

### 개발 시작
이제 django project 에 새로운 앱을 추가하고 page model 을 세팅하면 된다.   
여기부터는 위에 서술된 `wagtail 로 첫 프로젝트 시작하기` 내용과 동일하다.

`Note`
단, wagtail 프로젝트로 시작한 경우와 조금 다른 부분이 있다. wagatil 은 기본적인 `Page` 유형의 
초기 homepage 를 생성한다. 그런데 이 홈페이지에는 제목 이외의 컨텐츠 필드가 포함되어 있지 않다. 그런데 만약 기능을 가진 각자 자신만의 `HomePage` 클래스로 이를 바꾸고 싶을지도 모른다. 그렇게 하려면 wagtail 관리자 인터페이스에서 `Settings / Sites in the Wagtail admin` 부분의 site 데이터를 새로 만든 homepage 를 가리키도록 설정해야함을 유의해야 한다.




