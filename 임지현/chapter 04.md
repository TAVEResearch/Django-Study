# 목차
* [chapter 04](#chapter-04-프로젝트-첫-페이지-만들기)
  * [개발 코딩하기](#개발-코딩하기)
    * [URLconf 코딩하기](#urlconf-코딩하기)
    * [뷰 코딩하기](#뷰-코딩하기)

* [개념 정리](#개념-정리)
  * [✅ {% block %}](#--block-)
  * [✅ {% csrf_token %}](#--csrf_token-)
  * [✅ {% load static %}](#--load-static-)

<br><br>

---

<img src="https://user-images.githubusercontent.com/55045377/147868825-261a6cd1-1836-4c82-a5c7-e90d13643f5b.png" width=40%>

<br>

# chapter 04 프로젝트 첫 페이지 만들기
> ※ 첫 페이지는 사이트 전쳬의 이미지를 대표하므로, 기능보다는 디자인 측면이 중요하므로 HTML, 스타일시트, 자바스크립트 등의 지식이 필요한 분야이다.

## 개발 코딩하기
테이블 변경 사항이 없으므로 모델 코딩은 불필요하고, URLconf 코딩부터 시작한다.

### URLconf 코딩하기
애플리케이션에 대한 URL이 아니라 프로젝트에 대한 URL이므로 projects/urls.py 파일에 임포트 문장 및 루트(/) URL 두 줄만 추가하면 된다. 뷰 클래스는 HomeView, URL 패턴명은 'home'이라고 정의했다.
```python
from django.contrib import admin
from django.urls import path, include
from projects.views import HomeView    # 추가

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', HomeView.as_view(), name='home'),    # 추가
    path('bookmark/', include('bookmark.urls')),
    path('blog/', include('blog.urls')),
]
```

### 뷰 코딩하기
이제 URLconf에서 지정한 HomeView를 코딩한다. HomeView는 특별한 처리 로직 없이 단순히 템플릿만 보여주는 로직이므로, TemplateView 제네릭 뷰를 상속받아 코딩한다.

파일의 위치는 프로젝트와 관련된 뷰이므로, projects/views.py 파일에 코딩하는 것이 적절하다. 다음과 같은 내용으로 views.py 파일을 새로 만든다.
```python
from django.views.generic import TemplateView

class HomeView(TemplateView):                  # 1
    template_name = 'home.html'                # 2
```

위 소스를 라인별로 설명하면 다음과 같다.
* **# 1**: TemplateView 제네릭 뷰를 상속받아 사용하고 있다. TemplateView를 사용하는 경우에는 필수적으로 template_name 클래스 변수를 오버라이딩으로 지정해줘야 한다.<br>
TemplateView는 urls.py에서 view를 지정하지 않고 템플릿을 바로 지정해야 할 때 사용한다. 템플릿이 주어지면 해당 템플릿을 렌더링한다.
* **# 2**: projects 프로젝트의 첫 화면을 보여주기 위한 템플릿 파일을 home.html로 지정했다. 템플릿 파일이 위치하는 디렉토리 settings.py 파일의 TEMPLATE_DIRS 항목으로 지정되어 있다.

<br><br><br>





























---

# 개념 정리
## ✅ {% block %}
```python
{% block content %}
{% endblock %}
```
위 코드는 block을 만든 것과 같다. 템플릿 태그 {% block %}으로 HTML 내에 들어갈 수 있는 공간을 만든 것이다. 

<br>

## ✅ {% csrf_token %}
> Django의 CSRF공격에 대한 방어

### 1. CSRF(Cross Site Request Forgery)
웹사이트 취약점 공격의 하나로, 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위(수정, 삭제, 등록 등)를 특정 웹사이트에 요청하게 하는 공격이다.

### 2. Django의 CSRF공격에 대한 방어
CSRF 공격을 방어하기 위한 다양한 방법이 있지만 Django에서는 기본적으로 csrf token을 이용한다.

POST 요청에 대해서만 csrf token을 발급하고 체크한다. POST 양식을 사용하는 템플릿에서 \<form\> 태그 안에 {% csrf_token %} 태그를 사용.

### 3. 동작 과정
1. 사용자가 해당 페이지에 접속하면 Django에서 자동으로 csrf_token을 클라이언트로 보내어 cookie에 저장
2. 사용자가 form을 모두 입력한 후 제출버튼을 클릭한다.
3. form과 cookie의 csrf_token을 함께 POST로 전송한다.
4. 전송된 token의 유효성을 검증
5. 유효한 요청이면 요청을 처리
    * token이 유효하지 않거나(없거나 값이 잘못된 경우) 검증 오류 시에는 403 Forbidden Response 반환

### References
* https://chagokx2.tistory.com/49

<br>

## ✅ {% load static %}
### static 파일이란?
static 파일이란 js, css, image, font 등과 같이 개발자가 사전에 미리 서버에 저장 해둔 파일들을 말한다. 정적인 파일들이라고 할 수 있다.

### settings.py 에서의 설정
* **STATIC_URL**<br>
프로젝트 폴더의 settings.py 파일에서 최하단으로 내려보자.<br><br>
<img src="https://user-images.githubusercontent.com/55045377/147866522-430176d7-497e-463b-9fa3-9c00db5cb77e.png" width=50%><br>
STATIC_URL: static 파일이 제공되는 URL<br><br>
이미 settings 에 위와 같이 설정이 되어있다. 이를 통해 각 static 파일에 대한 URL의 고정값을 설정할 수 있다. 예시를 들자면 `{% static '경로' %}` 에 대해서 해당 URL 이 `STATIC_URL+'경로'` 로 바뀌게 되고 이는 다시 `'/static/경로'` 다음과 같이 바뀌게 되어 참조를 할 수 있다.

* **STATICFILES_DIRS**<br>
    ```python
    STATICFILES_DIRS = [
        BASE_DIR / 'static',
    ]
    ```
    STATICFILES_DIRS: static 파일 경로<br><br>
    추가로 지정을 해야하는 부분이다. 프로젝트 전반적으로 사용되는 static 경로가 어딘지 설정한다. <br>
    > ※ 참고로 static이라는 이름으로 폴더를 만들어야 장고에서 인식한다.
    
    <br>
    만약 프로젝트 폴더의 하위가 아닌, 앱 폴더 하위에 static 폴더를 관리하고 싶다면?<br>
    가령 config라는 폴더 안에 있는 static 폴더를 사용하고 싶다면 다음과 같이 작성해준다.
    
    ```python
    STATICFILES_DIRS = [
        BASE_DIR / 'static',
        os.path.join(BASE_DIR, 'config', 'static')
    ]
    ```
    config 내에 static 폴더를 추가로 만들었다. 이렇게 작성해주면 위 2경로에 있는 static 파일들을 가지고 온다.

* **STATIC_ROOT**<br>
    각 static 파일들은 각자 다른 경로에 나눠져 있다. 왜냐하면 프로젝트 전반적으로 사용하는 파일들은 STATICFILES_DIRS 에 담겨 있고, 각자의 app 안에는 app 에서 사용되는 파일들이 따로 모여있기 때문이다.<br>
    배포를 하기 위해서는 이들을 하나의 디렉토리에 모아야 하는데 아래 명령어로 한 번에 모을 수 있다.
    ```python
    python manage.py collectstatic
    ```
    하지만 어디로 모을지는 따로 지정을 해줘야 하는데, 그 경로가 바로 STATIC_ROOT 이다.<br>
    ```python
    STATIC_ROOT = os.path.join("staticfiles")
    ```
    STATIC_ROOT: static 복사 파일 경로
    > "staticfiles"라는 이름으로 설정한 이유는 collectstatic 명령어를 사용하면 staticfiles 폴더가 만들어지기 때문이다.

<br>
    
### templates 에서 사용
```html
{% load static %}
```
{% load static %}을 통해 settings.py에 작성한 static 파일들을 가져오게 된다. 즉, 준비한 static 파일들을 해당 html에 로드해달라는 의미이다.

이후에는 다음과 같이 img, css, js 등 여러 정적 파일들을 불러올 수 있다.
```html
{% static 'STATIC_URL 이후의 경로' %}
```

### References
* https://ssungkang.tistory.com/entry/Django-static-%ED%8C%8C%EC%9D%BC-%EB%8B%A4%EB%A3%A8%EA%B8%B0
* https://0ver-grow.tistory.com/912



















