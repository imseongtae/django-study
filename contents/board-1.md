# Board


## Table of Contents

  1. [BaseTemplate](#basetemplate)
  1. [Board Model](#board-model)
  1. [Board List View](#board-list-view)
  1. [Board List Template](#board-list-template)

---



## BaseTemplate
### MVT의 T를 확장하여 상속하기

`base.html` 생성 후 아래 내용 작성하기

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


## 


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

- `board_list.html` 생성 후 아래 내용 작성

{% raw %}
`{% for board in boards %}` 는  
for 문을 통해 `boards` 요소를 순회하여 `board`에 접근하는 코드이다.

페이징 기능을 만들 때 `{% if boards.has_previous %}` 또는 `{% if boards.has_next %}`를 통해 페이징을 할 수 없을 경우에 대해 분기처리를 주의해야 한다.

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


