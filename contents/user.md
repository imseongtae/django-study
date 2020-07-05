# USER


## Table of Contents

  1. [Settings](#Settings)
  1. [BaseTemplate](#BaseTemplate)
  1. [Register](#Register)
  1. [Register Model](#Register-Model)  
  1. [Register View](#Register-View)  
  1. [Register Template](#Register-Template)    
  1. [Login](#Login)  
  1. [Login View](#Login-View)
  1. [Login Forms](#Login-Forms)
  1. [Login Template](#Login-Template)  
  1. [Logout](#Logout)
  1. [Logout View](#Logout-View)

---

## Settings


### Add Application 
`settings.py`에서 사용할 애플리케이션 등록

```python 
INSTALLED_APPS = [
    '...'
    'board',
    'user',
]
```

### URL and View Mapping

`urls.py`에서 클라이언트의 요청을 분석하는 URLconf를 코딩, URL과 View를 매핑하는 작업  
'config'에서 작성

```python 
from django.contrib import admin
from django.urls import path, include
from user.views import home

urlpatterns = [
    path('admin/', admin.site.urls),
    path('user/', include('user.urls')),
    path('', home),
]
```

`application`에서 작성

```python 
from django.urls import path
from . import views

urlpatterns = [
    path('register/', views.register, name='register'),
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),

]
```

### admin 

관리자 페이지의 정보를 개선하는 두 가지 방법 중 하나(다른 한 가지는 model클래스에서 작성)로
리스트에서 더 많은 정보를 원할 때는 `admin.py`의 어드민 클래스에 명시할 수 있다. list_display 라는 필드를 통해 출력하고 싶은 필드를 선택할 수 있다.  
인자에 값을 넘겨서 명시하게 되면 클래스 객체가 리스트업이 되는 게 아니라 모델 클래스 안에 있는 필드들이 리스트업된다.
그래서 사용자명과 비밀번호가 나오게 되고, 보여지는 정보들이 개선된다.

```python
from django.contrib import admin
from .models import User

# Register your models here.
class UserAdmin(admin.ModelAdmin):
    list_display = ('username', 'password')

admin.site.register(User, UserAdmin)
```

### apps

`apps.py` 클래스에 아래의 내용을 기입

```python
from django.apps import AppConfig

class UserConfig(AppConfig):
    name = 'user'
```



**[⬆ back to top](#table-of-contents)**


## BaseTemplate
### MVT의 T를 확장하여 상속하기

디자인의 기준이 되는 HTML 파일을 만들어서
다른 HTML 파일에서 기준이 되는 파일을 상속받고 화면을 구현하는 기법

헤드 태그내의 내용을 상속 받는다. 
이를 통해 
1. 중복을 제거
1. 유지보수가 용이

### 상속하는 방법

1. children 파일에서 구현할 영역을 `{% block contents %}`와 `{% endblock %}`를 통해 설정

{% raw %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>Base</title>
    <!-- head 태그내에 Bootstrap 태그가 오면 된다. -->    
</head>
<body>
<!--  container  -->
<div class="container">
    {% block contents %}
    {% endblock %}
</div>
<!-- //container -->

</body>
</html>
```
{% endraw %}


### 상속받는 방법

1. `{% extends 'base.html' %}` 최상단에 작성
1. 시작 태그처럼 사용 `{% block contents %}`
1. 구현할 화면의 템플릿을 만들고
1. 닫는 태그처럼 사용 `{% endblock %}`

{% raw %}
```html
{% extends 'base.html' %}

{% block contents %}
<!--  Login  -->
<!--  block과 endblock 사이에 내용을 작성해주면 된다.  -->
<div class="row mt-5">
    <div class="col-12 text-center">
        <h1>로그인</h1>
    </div>
</div>
<!--  //Login  -->
{% endblock %}
```
{% endraw %}

**[⬆ back to top](#table-of-contents)**



## Register

## Register Model

문자열과 비밀번호는 문자열을 담을 수 있는 필드로 만들고,
데이트 타임의 약자로 registered_dttm 변수를 생성하여 DateTimeField 메소드를 호출하여 인자로 `auto_now_add=True`를 전달
auto_now_add 는 클래스가 저장되는 시점의 시간이 자동으로 저장되므로 현재 시간을 계산해서 넣어줄 필요가 없다.

`Meta` 클래스를 통해서 테이블 이름을 지정할 수 있다
테이블 이름을 지정하는 이유는 기본적으로 생성되는 앱들과 구분하기 위해서이다.

이후 `makemigrations`을 통해 변경사항을 정리하고, `migrate`를 통해 DB에 모델의 변경사항을 적용해야 함


```python
from django.db import models

# model에 작성된 클래스는 장고의 모델 클래스를 상속 받아야만 한다는 규칙을 가진다
class User(models.Model):
    username = models.CharField(max_length=32, verbose_name='사용자명')
    password = models.CharField(max_length=64, verbose_name='비밀번호')
    # EmailField는 데이터가 이메일 형태인지 검증까지 해줌
    useremail = models.EmailField(max_length=128, verbose_name='이메일')
    registered_dttm = models.DateTimeField(auto_now_add=True, verbose_name='등록일')

    # 첫 번째 방법, 두 가지의 방법이 있다고 한다.
    # 클래스가 문자열로 변환 되었을 때, 어떻게 변환할지 결정하는 함수가 있다.
    # username으로 반환하도록 변경을 한다. 이를 통해 관리자 페이지에 보여지는 정보를 개선할 수 있다.
    def __str__(self):
        return self.username

    class Meta:
        db_table = 'board_user'
        verbose_name = '사용자'
        verbose_name_plural = '사용자'
        # 장고는 모델을 보여줄 때 기본적으로 복수형을 보여주기 때문에
        # 복수형에 대한 설정을 따로 해주는 것이 좋다.
```

## Register View

url을 통해서 사용자 요청정보가 request 를 통해 들어온다. 이 때 레지스터로 들어오는 요청은 두 가지가 생긴다. 주소 URL로 들어오는 경우와 `Submit` 버튼을 눌러서 들어오는 경우이다.    


### URL을 통해서 들어오는 경우
URL을 통해서 들어오는 경우를 구분하기 위해서 request 정보의 method를 통한 정보를 전송하는 방식을 확인하고, 값이 ‘GET’이라면 render 단축함수의 인자로 request와 template을 전송한다.
	
### Submit 버튼을 통해서 들어오는 경우

request 정보의 method를 통해 들어온 값이 ‘POST’라고 한다면 render 단축함수의 인자로 request와 template 그리고 template으로 함께 전달할 정보를 전송한다  
input 태그를 통해 전달받은 사용자 데이터를 저장하기 위해서 input 태그의 Attribute 중 name 을 key값으로 하고, value를 가져온다.  
get('username', None) 메서드를 통해서 대상의 값이 없다면 None 을 저장하고, 아래쪽의 조건문을 통해서 데이터가 유효하게 입력되었는지 검증한다. 데이터를 검증한 후, 객체에 담아서 template에 전달하여 사용자의 입력값이 올바르지 않다면 그 처리를 반환한다. 

```python
from django.http import HttpResponse
from django.shortcuts import render, redirect
from django.contrib.auth.hashers import make_password, check_password
from .models import User
from .forms import LoginForm

def register(request):  
  if request.method == 'GET':
    # 경로는 템플릿 폴더를 바라보므로 경로를 따로 표현할 필요는 없다
    return render(request, 'register.html')
  elif request.method == 'POST':    
    username = request.POST.get('username', None)
    password = request.POST.get('password', None)
    re_password = request.POST.get('re-password', None)
    useremail = request.POST.get('useremail', None)
    # form 을 통해 넘어오는 input 속성의 name 값을 키로 사용해서 값을 얻는다.

    res_data = {}
    # 먼저 모든 값들이 들어와 있는지 확인을 하고 들어오지 않았을 때 아래 메시지 출력
    if not (username and password and re_password and useremail):
        res_data['info_error'] = '회원정보가 올바르게 입력되지 않았습니다.'
    elif password != re_password:
        res_data['password_error'] = '비밀번호가 다릅니다.'
    else:
      user = User(
        username=username,
        useremail=useremail,
        # 장고에서는 암호화해서 저장하는 make_password 메서드를 지원해줌
        # password=password가 아니라 아래처럼 더 개선된 방법으로 작성할 수 있다.
        password=make_password(password) # 비밀번호를 함수 안에 전달
      )
    # 이렇게 하고서 저장을 하면 끝이남
    user.save() # 클래스 변수 객체를 하나 생성하고 저장하면 된다..!

    return render(request, 'register.html', res_data)
```

## Register Template

{% raw %}
```html
{% extends 'base.html' %}

{% block contents %}
<div class="row mt-5">
    <div class="col-12 text-center">
        <h1>회원가입</h1>
    </div>
</div>

<div class="row mt-5">
    <div class="col-12">
        {{ info_error }}
    </div>
</div>

<div class="row mt-5">
    <div class="col-12">
        <form method="post" action=".">
          {% csrf_token %}
          <div class="form-group">
            <label for="username">사용자 이름</label>
            <input type="text" class="form-control" id="username" name="username" placeholder="사용자 이름">
          </div>
          <div class="form-group">
            <label for="password">비밀번호</label>
            <input type="password" class="form-control" placeholder="비밀번호" id="password" name="password">
                {{ password_error }}
          </div>
          <div class="form-group">
            <label for="re-password">비밀번호 확인</label>
            <input type="password" class="form-control" placeholder="비밀번호 확인" id="re-password" name="re-password">
                {{ password_error }}
          </div>
          <div class="form-group">
            <label for="useremail">이메일 주소</label>
            <input type="email" class="form-control" placeholder="이메일 주소" id="useremail" name="useremail">
          </div>

          <button type="submit" class="btn btn-primary">등록</button>
        </form>
    </div>
</div>

{% endblock %}
```
{% endraw %}

**[⬆ back to top](#table-of-contents)**



## Login

## Login Model

Login Model은 reigister 모델을 그대로 사용  
views.py에 아래 함수 작성

## Login View

```python
def login(request):
    if request.method == 'POST':
        # POST일때는 폼 클래스 변수를 만들 때 Post 데이터를 넣어준다.
        form = LoginForm(request.POST)
        if form.is_valid():
            # session 코드가 위치한다.
            request.session['user'] = form.user_id
            return redirect('/')
    else:
        form = LoginForm()

    return render(request, 'login.html', {'form': form})
```

## Login Forms

Django에서 제공하는 `form` 객체를 사용하면 view의 코드량을 줄이고 직관적으로 개선할 수 있다.  
user app폴더에 `forms.py`라는 파일을 만들고, 아래 `LoginForm`클래스에서 데이터를 검증한다.


```python
from django import forms
from .models import User
from django.contrib.auth.hashers import check_password

class LoginForm(forms.Form):
    username = forms.CharField(
        error_messages={
            'required': '아이디를 입력해주세요'
        },
        max_length=32, label="사용자 이름")
    password = forms.CharField(
        error_messages={
            'required': '비밀번호를 입력해주세요'
        },
        widget=forms.PasswordInput, label="비밀번호")
    # 비밀번호를 패스워드 타입으로 만들기 위해선 위젯을 직접 지정해야 한다.

    # 비밀번호 일치여부 확인하는 코드
    def clean(self):
        cleaned_data = super().clean() # 값이 들어있지 않으면 여기서 실패처리해서 나간다.
        # 값이 들어왔다는 검증이 끝나면 값을 가지고 온 후에
        username = cleaned_data.get('username')
        password = cleaned_data.get('password')

        if username and password:
			try:
            	user = User.objects.get(username=username)
			except User.DoesNotExist:			
				self.add_error('username', '아이디가 없습니다.')
                return # 예외 이후 코드의 진행을 return을 통해 막음
				
            if not check_password(password, user.password):
                self.add_error('password', '비밀번호가 틀렸습니다.') # 특정 필드에 에러를 넣는 함수
            else:
                self.user_id = user.id # 이렇게 하면 self를 통해서 클래스 변수로 들어가고 밖에서도 접근할 수 있게 된다.

```

## Login Template


장고에서 관리해주는 `form` 객체는 form에서 필요한 것들을 기본적으로 만들어준다.
`form` 객체의 태그는 장고가 자동으로 생성해주고, 사용자의 입력값을 자동으로 검증해준다.

커스터 마이징이 가능한 것인가..!?라는 의문이 들었지만 커스터 마이징이 가능함!
> 아직 어느 정도까지 커스터마이징이 가능한지는 모름
`as_p` 를 하면 항목 하나하나를 p태그로 감쌈
마찬가지로 `as_table` 등 여러가지 태그로 감쌀 수 있다

{% raw %}
```html
{% extends 'base.html' %}

{% block contents %}
<!--  Login  -->
<div class="row mt-5">
    <div class="col-12 text-center">
        <h1>로그인</h1>
    </div>
</div>

<div class="row mt-5">
    <div class="col-12">
        {{ info_error }}
        {{ password_error }}
        {{ password_valid }}
    </div>
</div>

<div class="row mt-5">
    <div class="col-12">
        <form method="post" action=".">
            {% csrf_token %}
            {% for field in form %}
                <div class="form-group">
                    <label for="{{ field.id_for_label }}">{{ field.label }}</label>
                    <input type="{{ field.field.widget.input_type }}" class="form-control" id="{{ field.id_for_label }}"
                           placeholder="{{ field.label }}" name="{{ field.name }}" />
                </div>
                {% if field.errors %}
                    <span style="color: red">{{ field.errors }}</span>
                {% endif %}
            {% endfor %}
          <button type="submit" class="btn btn-primary">로그인</button>
        </form>
    </div>
</div>
<!--  //Login  -->
{% endblock %}
```
{% endraw %}

**[⬆ back to top](#table-of-contents)**

## Logout

## Logout View

Logout은 Login의 Template을 그대로 사용하고, View는 주소만 지정해주면 된다.

```python
def logout(request):

    if request.session.get('user'):
        del(request.session['user'])
    return redirect('/')
    # 템플릿이 필요가 없다. 주소만 지정해주면 된다.


```



**[⬆ back to top](#table-of-contents)**





