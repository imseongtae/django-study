# Board


## Table of Contents

  1. [Settings](#Settings)
  1. [BaseTemplate](#BaseTemplate)
  1. [Board Model](#Board-Model)
  1. [Board List View](#Board-List-View)
  1. [Board List Template](#Board-List-Template)
  1. [Board Write View](#Board-Write-View)
  1. [Board Detail View](#Board-Detail-View)  
  1. [Board Write Template](#Board-Write-Template)
  1. [Board Detail Template](#Board-Detail-Template)
  1. [Link Connect](#Link-Connect)

  

---

## Settings

### Folder structure

```
<Root folder>
│
├<community_project>
│ ├── config
│ ├── user
│ ├── board
│ ├── db.sqlite3
│ └── manage.py
└venv (가상환경 설정)
```

### Add Application 
`settings.py`에서 사용할 애플리케이션 등록

```python
INSTALLED_APPS = [
    'board',
]
```

### URL and View Mapping

```python
from django.contrib import admin
from django.urls import path, include
from user.views import home

urlpatterns = [
    path('board/', include('board.urls')),
]
```

```python
from django.urls import path
from . import views

urlpatterns = [
    path('list', views.board_list, name='board_list'),
]
```

### admin 

`list_display`필드에는 타이틀만 나오게끔 설정  
그리고 보드어드민 연결

```python
from django.contrib import admin
from .models import Board

# Register your models here.
class BoardAdmin(admin.ModelAdmin):
    list_display = ('title', )

admin.site.register(Board, BoardAdmin) 
```

### apps

자동으로 등록이 되는 것 같은데?


### Static Files
`static` 폴더를 생성하고 그 안에 `CSS, JavaScript` 파일을 정리한다. 
`STATIC_URL`은 파일에 접근했을 때 어느 폴더에 있는지 알려주기 위한 변수값
`STATICFILES_DIRS` 안에 리스트로 경로를 넣어주면 된다.
`BASE_DIR`은 기본 프로젝트를 가리킨다.

```
<Root folder>
│
├<community_project>
│ └── static
└venv (가상환경 설정)
```

```python
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.0/howto/static-files/
STATIC_URL = '/static/'

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]
```


**[⬆ back to top](#table-of-contents)**


## BaseTemplate
### MVT의 T를 확장하여 상속하기

{% raw %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>Base</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
<!--    <link rel="stylesheet" href="/static/bootstrap.min.css" />-->
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>

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

**[⬆ back to top](#table-of-contents)**


## Board Model

writer 필드에는 ForeignKey를 통해 User 테이블과 연결한 값을 넣는다.
ForeignKey 메소드의 인자값으로 전달하는 `on_delete`는 사용자가 작성한 글들에 대한 정책을 설정할 수 있다.
`on_delete`의 값으로 `CASCADE`를 전달하면 user가 삭제되었을 때 user가 작성한 글들도 함께 삭제된다. 

```python
from django.db import models

class Board(models.Model):
    title = models.CharField(max_length=128, verbose_name='제목')
    contents = models.TextField(verbose_name='내용') # 길이에 제한이 없음
    writer = models.ForeignKey('user.User', on_delete=models.CASCADE, verbose_name='작성자')

    registered_dttm = models.DateTimeField(auto_now_add=True, verbose_name='등록시간')

    def __str__(self):
        return self.title

    class Meta:
        db_table = 'board'
        verbose_name = '게시글'
        verbose_name_plural = '게시글'
```


**[⬆ back to top](#table-of-contents)**


## Board List View

`boards` 필드를 통해서 board 를 가져온 다음 단축함수 render를 통해 Template 으로 전달한다.
`Board.objects.all().order_by`의 메서드로 dash(-)가 포함된 값을 전달하면 역순 정렬(내림차순)을 의미한다. 
`page`변수에는 페이지 번호를 get형태로 받고, p라는 값으로 받는 데 없다면 첫 번째 페이지로 지정
그리고 `page`를 int형으로 바꾼다.
Paginator 생성자의 인자로 한 페이지에 나오는 페이지의 수를 설정한 후에 이를 `paginator`변수에 담는다.
`paginator` 참조(클래스) 변수를 이용해서 화면에 보여줄 수 있다. 

`boards`에는 Paginator를 통해 얻은 페이지에 대한 정보 들어있음

```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Board

# Create your views here.
def board_list(request):
    all_boards = Board.objects.all().order_by('-id') 
    
    page = int(request.GET.get('p', 1))
    paginator = Paginator(all_boards, 2)
    
    borads = paginator.get_page(page)        
    return render(request, 'board_list.html', {'boards': boards})
```

**[⬆ back to top](#table-of-contents)**



## Board List Template

`{% for board in boards %}` 는  
for 문을 통해 `boards` 요소를 순회하여 `board`에 접근하는 코드이다.

페이징 기능을 만들 때 `{% if boards.has_previous %}` 또는 `{% if boards.has_next %}`를 통해 페이징을 할 수 없을 경우에 대해 분기처리를 주의해야 한다.

{% raw %}
```html
{% extends "base.html" %}

{% block contents %}

<div class="row mt-5">
  <div class="col-12">
    <table class="table table-light">
      <thead class="thead-light">
        <tr>
          <th>#</th>
          <th>제목</th>
          <th>아이디</th>
          <th>일시</th>
        </tr>
      </thead>
      <tbody class="text-dark">
        {% for board in boards %}
        <tr>
          <td>{{ board.id }}</td>
          <td>{{ board.title }}</td>
          <td>{{ board.writer }}</td>
          <td>{{ board.registered_dttm }}</td>
        </tr>
        {% endfor %}
      </tbody>
    </table>
  </div>
</div>

<div class="row">
  <div class="col-12">
    <nav>
      <ul class="pagination justify-content-center">
          {% if boards.has_previous %}
          <li class="page-item">
              <a class="page-link" href="?p={{ boards.previous_page_number }}">prev</a>
          <li>
          {% else %}
          <li class="page-item disabled">
              <a class="page-link" href="#">prev</a>
          <li>
          {% endif %}

          <li class="page-item active">
              <a class="page-link" href="#">{{ boards.number }} / {{ boards.paginator.num_pages }}</a>
          </li>

          {% if boards.has_next %}
          <li class="page-item">
              <a class="page-link" href="?p={{ boards.next_page_number }}">next</a>
          <li>
          {% else %}
          <li class="page-item disabled">
              <a class="page-link" href="#">next</a>
          </li>
          {% endif %}
      </ul>
    </nav>
  </div>
</div>

<div class="row">
    <div class="col-12">
        <button class="btn btn-primary">글쓰기</button>
    </div>
</div>

{% endblock %}

```
{% endraw %}

**[⬆ back to top](#table-of-contents)**



## Board Write View
로그인과 비슷한 점이 있음.  
상세보기로 넘길지 List로 넘길지 글을 작성 이후에 결정할 수 있음  
로그인하지 않은 사용자가 글을 작성하려고 할 때 예외처리를 해야 하므로  
request 메서드를 확인하기 전에 일단 사용자가 있는지 먼저 확인

```python
from django.shortcuts import render, redirect
from django.http import Http404
from user.models import User
from .models import Board
from .forms import BoardForm

def board_write(request):
  if not request.session.get('user'):
    return redirect('/user/login')

  if request.method == 'POST':
    form = BoardForm(request.POST) # POST일 때 데이터를 넣는다.
    if form.is_valid():
      # 세션에서 유저 아이디를 가져오고
      user_id = request.session.get('user');
      # 유저아이디를 이용해 모델에서 pk가 user_id인 것을 가져옴
      user = User.objects.get(pk=user_id)

      board = Board()
      board.title = form.cleaned_data['title']
      board.contents = form.cleaned_data['contents']
      # 사용자가 로그인했으면 정보가 session에 들어있다.
      board.writer = user # user 를 writer에 추가
      board.save() # save를 통해 데이터베이스에 저장이 된다.

      # redirect 경로
      return redirect('/board/list/')
  else:
    form = BoardForm()
  return render(request, 'board_write.html', {'form': form})
```


## Board Detail View

상세보기를 통해 확인하는 현재 글이 몇 번째 글인지 구별할 수 있어야 함  
이를 위해 pk을 board_detail 함수의 인자값으로 받고 `Board.objects.get` 메소드의 인자로 넘김

1. 로그인하지 않았을 경우에 대한 처리를 해야 함(board_write에서 작성)
1. 없는 번호의 게시글을 입력할 경우 예외처리(board_detail에서 작성)

```python
def board_detail(request, pk):
  try:
    board = Board.objects.get(pk=pk)
  except Board.DoesNotExist:
    raise Http404('게시글을 찾을 수 없습니다.')
    
  return render(request, 'board_detail.html', {'board': board})
```


**[⬆ back to top](#table-of-contents)**





## Board Write Template

{% raw %}
```html
{% extends "base.html" %}

{% block contents %}

<div class="row mt-5">
  <div class="col-12">
    <form method="post" action=".">
      {% csrf_token %}
      {% for field in form %}
        <div class="form-group">
          <label for="{{ field.id_for_label }}">{{ field.label }}</label>
          {{ field.field.widget.name }}
          <!--  새로운 태그를 만들어야 한다.-->
          {% ifequal field.name 'contents' %}
              <textarea class="form-control" name="{{ field.name }}" placeholder="{{ field.label }}"></textarea>
          {% else %}
          <input type="{{ field.field.widget.input_type }}" class="form-control" id="{{ field.id_for_label }}"
                 placeholder="{{ field.label }}" name="{{ field.name }}" />
          {% endifequal %}
        </div>
      
        {% if field.errors %}
            <span style="color: red">{{ field.errors }}</span>
        {% endif %}
      {% endfor %}
      <button type="submit" class="btn btn-primary">글쓰기</button>
    </form>
    
  </div>
</div>

{% endblock %}
```
{% endraw %}

## Board Detail Template
작성한 글의 내용을 확인하는 Board Detail Template

{% raw %}
```html
{% extends "base.html" %}

{% block contents %}

<div class="row mt-5">
  <div class="col-12">
    <div class="form-group">
      <label for="title">제목</label>
      <input type="text" class="form-control" id="title" value="{{ board.title }}" readonly />
      <label for="contents">내용</label>
      <textarea class="form-control" readonly>{{ board.contents }}</textarea>
    </div>

    <button class="btn btn-primary">돌아가기</button>
  </div>
</div>

{% endblock %}
```
{% endraw %}



**[⬆ back to top](#table-of-contents)**



## Link Connect

## Home View

User home view에서 단축함수를 사용하여 `home.html`을 전달
session에 대한 처리를 Home Template에게 위임함!

```python
def home(request):
  # user_id = request.session.get('user')
  # if user_id:
  #    fcuser = Fcuser.objects.get(pk=user_id)
  return render(request, 'home.html');
```

### Home Template

{% raw %}
```html
{% extends 'base.html' %}

{% block contents %}
<!--  Home  -->
<div class="row mt-5">
  <div class="col-12 text-center">
    <h1>Home Page</h1>
  </div>
</div>

<div class="row mt-5">
  {% if request.session.user %}
    <div class="col-12">
      <button class="btn btn-primary btn-block" onclick="location.href='/fcuser/logout/'">Logout</button>
    </div>
  {% else %}
    <div class="col-6">
      <button class="btn btn-primary btn-block" onclick="location.href='/fcuser/login/'">Login</button>
    </div>
    <div class="col-6">
      <button class="btn btn-primary btn-block" onclick="location.href='/fcuser/register/'">Sign in</button>
    </div>
  {% endif %}
</div>

<div class="row mt-1">
  <div class="col-12">
    <button class="btn btn-primary btn-block" onclick="location.href='/board/list/'">View Post</button>
  </div>
</div>
<!--  //Home  -->
{% endblock %}
```
{% endraw %}


### Board List Template

`tr` 태그의 attribute에 `onclick="location.href`속성을 이용하여 각 게시글의 경로를 연결

{% raw %}
```html
<tbody class="text-dark">
<!--  for 문을 통해 요소를 순회  -->
  {% for board in boards %}
    <tr onclick="location.href='/board/detail/{{ board.id }}'">
      <td>{{ board.id }}</td>
      <td>{{ board.title }}</td>
      <td>{{ board.writer }}</td>
      <td>{{ board.registered_dttm }}</td>
    </tr>
  {% endfor %}
</tbody>
```
{% endraw %}

게시판 하단에 글쓰기 화면으로 넘어갈 수 있도록`onclick`을 통해서 `href` 경로 추가

```html
<div class="row">
  <div class="col-12">
    <button class="btn btn-primary" onclick="location.href='/board/write/'">글쓰기</button>
  </div>
</div>
```

### Board Write Template

`form` 태그 안에 게시판 목록의 경로를 가진 돌아가기 버튼을 만듦

```html
<form method="post" action=".">
  <button type="button" class="btn btn-primary" onclick="location.href='/board/list/'">돌아가기</button>
</form>
```

### Board Detail Template

게시판 목록으로 이동할 수 있도록 `onclick="location.href`의 attribute를 사용하여 돌아가기 버튼에 경로를 추가

```html
<button type="button" class="btn btn-primary" onclick="location.href='/board/list/'">돌아가기</button>
```


**[⬆ back to top](#table-of-contents)**

