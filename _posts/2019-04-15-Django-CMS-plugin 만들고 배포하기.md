---
tags: Django, Django-CMS
title: 2019-04-15-Django-CMS-plugin 만들고 패키지 배포하기
---
# 들어가기전
Django CMS 라이브러리의 플러그인을 만들어보고 이를 패키징하여 배포해보자.       
이를 위해서는 크게 두가지의 개념을 이해해야한다.    

> [Django-CMS-플러그인 만들기](http://docs.django-cms.org/en/latest/how_to/custom_plugins.html)      
> [Django 재사용 가능한 앱을 만드는 법](https://docs.djangoproject.com/ko/2.2/intro/reusable-apps/)
  
Django CMS 플러그인을 만들 수 있어야하고 동시에 Django 앱을 배포하는 방법도 알아야한다.

# Django CMS 플러그인 만들기
## Django CMS 플러그인을 왜 만들어야 하는가?
Django CMS 플러그인은 다른 장고 앱에서의 컨텐츠를 django-CMS 페이지로 통합하는 가장 편리한 방법이다. 예를 들어, 당신이 Django CMS 를 사용해서 음반 회사의 사이트를 개발하고 있다고 가정해보자.
아마 당신은 당신의 홈페이지에 `최신 음원` 박스를 가지고 있을 것이다.      

물론 당신은 그 페이지를 직접 자주 수정하거나 정보들을 자주 업데이트 할지도 모른다. 그런데 센스있는 회사라면 그 카탈로그를 이번주의 최신 음반이 뭔지를 잘 알고있는 장고로 관리할 것이다.   

이렇게 하는 것은 당신을 엄청 편하게 할 수 있는 훌륭한 기회이다. 당신이 할 일은 그냥 django-CMS-plugin 을 만들고 이를 당신의 홈페이지에 추가만 하면된다. 그러면 당신은 최신 음반 정보들을 당신의 웹사이트에 직접 퍼블리싱 하는 작업을 하지 않아도 된다.

플러그인은 재사용 가능하다. 아마도 당신의 음반 회사는 스위스 펑크 음반을 그 앨범에 대한 웹사이트에다가 연속적으로 재발행 할지도 모른다. 이럴 때, 당신은 그 페이지에서 같은 플러그인을 추가하고 약간 다르게 설정하기만 하면, 최신 음반 정보를 발행할 수 있을 것이다.

## 개요
Django CMS 플러그인은 기본적으로 3가지로 구성되어있다.      
* `Editor` 플러그인 : 플러그인이 배포될 때마다 그 플러그인을 구성하는 역할
* `Publisher` 플러그인 : 무엇을 (페이지에)발행할지 결정하는 자동적인 작업을 하는 역할
* `Template` 플러그인 : 웹페이지에 정보를 렌더링하는 역할

이는 장고의 MTV (모델-템플릿-뷰) 스키마와 일치한다.
* `Editor` = `Model`
* `Publisher` = `View`
* `Template` = `Template`

그리고 플러그인은 아래의 요소를 가지고 구축한다.

* 플러그인 인스턴스의 구성을 저장하는 cms.models.pluginModel.CMSPlugin 의 하위 클래스
* 플러그인의 작동 논리를 결정하는 cms.plugin_base.CMSPluginBase 의 하위클래스
* 플러그인을 렌더하는 템플릿

### cms.plugin_base.CMSPluginBase
`cms.plugin_base.CMSPluginBase` 은 사실 `django.contrib.admin.ModelAdmin` 의 하위클래스이다.

CMSPluginBase 가 ModelAdmin 의 하위클래스라는 점에서, CMS 플러그인 개발자도 몇 가지 중요한 ModelAdmin 옵션을 사용할 수 있다.

* `exclude`
* `fields`
* `fieldsets`
* `form`
* `formfield_overrides`
* `inlines`
* `radio_fields`
* `raw_id_fields`
* `readonly_fields`

그런데 모든 ModelAdmin 옵션들이 CMS 플러그인에 효과적인 것은 아니라는 것을 유의해야한다.
특히, ModelAdmin 의 `changelist`에 의해 사용되는 전용 옵션들 모두는 효과가 없을 것이다. 
CMS 에서는 사용되지 않으면서 주목할만한 옵션들은 다음과 같다:

* `actions`
* `actions_on_top`
* `actions_on_bottom`
* `actions_selection_counter`
* `date_hierarchy`
* `list_display`
* `list_display_links`
* `list_editable`
* `list_filter`
* `list_max_show_all`
* `list_per_page`
* `ordering`
* `paginator`
* `preserve_fields`
* `save_as`
* `save_on_top`
* `search_fields`
* `show_full_result_count`
* `view_on_site`

### 모델과 구성에 관한 점은 제외
플러그인의 모델에 해당하는 cms.models.pluginmodel.CMSPlugin 의 하위클래스는 사실 선택사항이다.

단 한가지의 일만 하는 플러그인의 경우에는 아마 별도의 플러그인 구성이 필요하지 않을 것이다.
(다만, 기능이 복잡해지는 플러그인의 경우에는 별도의 구성이 필요해지고 이에 따라 모델도 필요하게 될 것이다.)

예를 들어, 지난 7일 동안의 최고 판매 기록에 대한 정보만 게시하는 플러그인이 있다고 가정해보자. 이는 분명히 매우 유연하지 못할 것이다. 왜냐하면 당신이 같은 플러그인을 지난 달 베스트셀러 부분에서 재사용 할 수 없기 때문이다.

보통 플러그인을 구성할 수 있는 것이 유용하다는 것을 알게 되면, 모델이 필요하게 될 것이다.

## 간단한 플러그인 만들기
당신은 아마도 `python manage.py startapp` 을 사용해서 당신의 플러그인 앱에 대한 기본적인 레이아웃을 잡을 것이다. (`INSTALLED_APPS` 에 플러그인을 추가해야함)
(여기까지는 기본 Django APP 을 새로 만드는 과정과 동일하다)

여기에 더해서 이렇게 만든 Django App 에  `cms_plugins.py` 파일 하나만 추가하면 된다.

`cms_plugins.py` 파일 안에 만든 플러그인을 넣어야 한다. 코드 예시:
``` python
from cms.plugin_base import CMSPluginBase
from cms.plugin_pool import plugin_pool
from cms.models.pluginmodel import CMSPlugin
from django.utils.translation import ugettext_lazy as _

@plugin_pool.register_plugin
class HelloPlugin(CMSPluginBase):
    model = CMSPlugin
    render_template = "hello_plugin.html"
    cache = False
```

이제 template 만 추가해주면 끝난다. 루트 템플릿 디렉토리에 아래의 코드가 작성된 `hello_plugin.html` 파일 하나만 추가해주면 된다.

``` html
{% raw %}
<h1>Hello {% if request.user.is_authenticated %}{{ request.user.first_name }} {{ request.user.last_name}}{% else %}Guest{% endif %}</h1>
{% endraw %}
```

이 플러그인은 로그인한 유저 또는 로그인 하지 않았을 경우 게스트를 환영하는 아주 간단한 hello_world 플러그인이다.

이제 위의 코드들에 대해 자세히 들여다보자
`cms.plugins.py` 파일은 `cms.plugin_base.CMSPluginBase` 의 하위클래스를 정의해야하는 곳이다. 그리고 이 클래스들은 서로 다른 플러그인들을 정의한다.

아래는 플러그인 클래스에서 필요한 두 가지 속성들이다:
* `model` : 이 플러그인에 대한 정보를 저장하는데 사용할 모델이다. 만약 모델에 저장될 어떠한 특별한 정보를 요구하지 않는다면 (이를테면, 환경 설정=`configuration` 과 같은 정보) `cms.models.pluginmodel.CMSPlugin` 을 간단히 사용할 수 있다. (이 모델을 조금 더 자세히 알아볼것이다) 일반적인 관리자 클래스에서 이 정보를 제공할 필요가 없다. 왜냐하면 `admin.site.register(Model, Admin)` 에서 관리하지만, 플러그인은 그런 방식으로 등록되지 않기 때문이다.
* `name` : `admin` 에서 보여질 플러그인의 이름이다. 일반적으로 이는 `django.utils.translation.ugettext_lazy()` 를 사용하여 번역가능한 언어로 쓰는 것이 좋지만 이는 선택사항이다. 기본적으로 이 이름은 클래스 이름의 더 좋은 버전이다.

그리고 아래는 만약 `render_plugin` 속성이 `True` 일 때, 정의 해야만 하는 것들이다:
* `render_template` : 이 플러그인을 렌더링할 템플릿
or
* `get_render_template` : 플러그인을 렌더링할 템플릿 경로를 리턴하는 메서드.

이러한 속성들과 더불어 `render()` 메서드 역시 override 할 수 있다. 이 메서드를 오버라이드 해서 플러그인을 렌더링하기 위해 사용되는 template context 변수들을 정해줄 수 있다. 기본적으로 이 메서드는 context 에 `instance` 와 `placeholder` 객체만 추가해주면 되지만, 플러그인이 필요한 context 를 포함하도록 오버라이드 할 수 있다.

다른 많은 메서드들도 당신의 CMSPluginBase 하위클래스를 (플러그인 클래스) 오버라이딩 하기 위해 사용할 수 있다. 참조: [CMSPluginBase - 자세히 보기](http://docs.django-cms.org/en/latest/reference/plugins.html#cms.plugin_base.CMSPluginBase)



