# Board 두 번째


## Table of Contents

  1. [Board Write View](#Board-Write-View)
  1. [Board Detail View](#Board-Detail-View)  
  1. [Board Write Template](#Board-Write-Template)
  1. [Board Detail Template](#board-detail-template)

---



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

