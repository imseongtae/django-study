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


