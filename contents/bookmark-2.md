# bookmark - 2

## Template



### bookmark_confirm_delete.html
bookmark/templates/bookmark의 경로 아래
`bookmark_confirm_delete.html` 생성

{% raw %}
```html
{% extends 'Base.html' %}

{% block title %}Delete{% endblock %}


{% block content %}
<form action="" method="post">
    {% csrf_token %}
    <div class="alert alert-danger">
        삭제할까요? {{ object }}
    </div>
    <input type="submit" value="삭제" class="btn btn-danger">
</form>

{% endblock %}
```
{% endraw %}


### bookmark_detail.html
bookmark/templates/bookmark의 경로 아래

`bookmark_detail.html` 생성

{% raw %}
```html
{% extends 'Base.html' %}

{% block title %}Delete{% endblock %}


{% block content %}
    {{ object.site_name }} <br>
    {{ object.url }}

{% endblock %}
```
{% endraw %}


### bookmark_form.html
bookmark/templates/bookmark의 경로 아래  
`bookmark_form.html` 생성

{% raw %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>bookmark_form</title>

       <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>


</head>
<body>
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <a class="navbar-brand" href="#">Bookmark</a>
  <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
    <span class="navbar-toggler-icon"></span>
  </button>
  <div class="collapse navbar-collapse" id="navbarSupportedContent">
    <ul class="navbar-nav mr-auto">
      <li class="nav-item active">
        <a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
      </li>
    </ul>
  </div>
</nav>


  <form action="" method="post">
      {% csrf_token %}
      {{ form.as_p }}
      <input type="submit" value="등록" class="btn btn-info">

  </form>

</body>
</html>
```
{% endraw %}


### bookmark_list.html

bookmark/templates/bookmark의 경로 아래  
`bookmark_list.html` 생성

{% raw %}
```html
{% extends 'Base.html' %}

{% block title %}Delete{% endblock %}

{% block content %}


<div class="btn-group">
    <a href="add/" class="btn btn-info">북마크 추가</a>
</div>

<table class="table table-dark">
    <thead>
        <tr>
            <th scope="col">#</th>
            <th scope="col">Site</th>
            <th scope="col">URL</th>
            <th scope="col">Modify</th>
            <th scope="col">Delete</th>
        </tr>
    </thead>
    <tbody>
<!--    이름을 지정 안 하면  object_list 가 날라온다. Default -->
        {% for bookmark in object_list %}
            <tr>
                <td>{{ forloop.counter }}</td>
                <td><a href="detail/{{ bookmark.id }}">{{ bookmark.site_name }}</a></td>
                <td><a href="{{ bookmark.url }}">{{ bookmark.url }}</a></td>
                <td><a href="_update/{{ bookmark.id }}" class="btn btn-success">수정</a></td>
                <td><a href="delete/{{ bookmark.id }}" class="btn btn-danger">삭제</a></td>
                <!--   1,2,3 을 나오게 하려구-->
            </tr>
        {% endfor %}

    </tbody>

</table>
{% if is_paginated %}
       <ul class="pagination justify-content-center pagination-sm">
           {% if page_obj.has_previous %}
               <li class="page-item">
                   <a class="page-link" href="{% url 'list' %}?page={{ page_obj.previous_page_number }}" tabindex="-1">Previous</a>
               </li>
           {% else %}
               <li class="page-item disabled">
                   <a class="page-link" href="#" tabindex="-1">Previous</a>
               </li>
           {% endif %}
            <!--   페이지 숫자 만큼 계속 루프를 돌면서..!         -->
           {% for object in page_obj.paginator.page_range %}
               <li class="page-item {% if page_obj.number == forloop.counter %}disabled{% endif %}">
                   <a class="page-link" href="{{ request.path }}?page={{ forloop.counter }}">{{ forloop.counter }}</a>
               </li>
           {% endfor %}
           {% if page_obj.has_next %}
               <li class="page-item">
                   <a class="page-link" href="{% url 'list' %}?page={{ page_obj.next_page_number }}">Next</a>
               </li>
           {% else %}
               <li class="page-item disabled">
                   <a class="page-link" href="#">Next</a>
               </li>
           {% endif %}
       </ul>
{% endif %}


{% endblock %}
```
{% endraw %}


### bookmark_update.html
bookmark/templates/bookmark의 경로 아래  
`bookmark_update.html`

{% raw %}
```python
{% extends 'Base.html' %}

{% block title %}수정{% endblock %}


{% block content %}
     <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <input type="submit" value="수정" class="btn btn-info">
    </form>

{% endblock %}
```
{% endraw %}


