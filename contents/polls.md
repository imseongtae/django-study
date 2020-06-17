# Polls


## Table of Contents

  1. [Settings](#Settings)
  1. [Index View](#Index-View)
  1. [Index Template](#Index-Template)
  1. [Detail View](#Detail-View)
  1. [Detail Template](#Detail-Template)
  1. [Vote View](#Vote-View)
  1. [Results View](#Results-View)
  1. [Results Template](#Results-Template)

---

## Settings

### Add Application 
`settings.py`에서 사용할 애플리케이션 등록

```python
INSTALLED_APPS = [
    ...
    'polls.apps.PollsConfig',
]
```

### Timezone

```python
TIME_ZONE = 'Asia/Seoul'
```

### URL and View Mapping

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('polls/', include('polls.urls')),
]
```

`polls/urls.py` 에서는 `app_name`를 통해 URL 패턴의 이름이 충돌생기는 것을 방지  
`app_name`을 통해서 `polls` 애플리케이션에서 사용하는 패턴이름임을 명확히 할 수 있다.

```python
from django.urls import path
from . import views

app_name = 'polls'
urlpatterns = [
  path('', views.index, name="index"),
  path('<int:question_id>/', views.detail, name="detail"),
  path('<int:question_id>/results', views.results, name="results"),
  path('<int:question_id>/vote', views.vote, name="vote"),
]
```

### admin 

```python
from django.contrib import admin
from .models import Question, Choice

# Register your models here.
admin.site.register(Question)
admin.site.register(Choice) 
```

## Index View
`render`단축함수를 통해 html 파일에 `context` 변수를 담아서 HTML 텍스트를 만들고, 이를 HttpResponse 객체를 통해 반환

```python
def index(request):
  latest_question_list = Question.objects.all().order_by('-pub_date')[:5]
  context = {'latest_question_list': latest_question_list}
  return render(request, 'polls/index.html', context)
```

## Index Template 
템플릿에서 필요한 `Question 객체`의 리스트가 들어있는 정보는   
`latest_question_list` 변수를 통해 `views.index`로부터 넘겨받는다.

```html
{% if latest_question_list %}
  <ul>
    {% for question in latest_question_list %}
      <li>
        <a href="/polls/{{ question.id }}">{{ question.question_text }}</a>
      </li>
    {% endfor %}
  </ul>
{% else %}
  <p>
    <strong>No Polls are available!</strong>
  </p>
{% endif %}
```


**[⬆ back to top](#table-of-contents)**


## Detail View

`get_object_or_404` 함수로 넘겨받는 첫 번째 인자는 모델 클래스, 두 번째 인자는 검색조건이다.   
`Question`모델 클래스의 `pk`가 `question_id`인 검색 조건에 맞는 객체를 조회하여, 조건에 맞는 객체가 없다면 Http404 예외를 발생  
URL패턴에서 정규표현식으로 추출한 `question_id` 파리미터가 뷰함수의 인자로 넘어옴

```python
def detail(request, question_id):
  question = get_object_or_404(Question, pk=question_id)
  return render(request, 'polls/detail.html', {'question': question})
```

## Detail Template 

`choice_set`은 1:N 관계에서 1 테이블에 연결된 N테이블의 항목들이란 의미로
`xxx_set` 속성을 제공한다. `Question` 테이블의 `question 레코드`에 연결된
`Choice 테이블의 레코드` 모두를 뜻한다. 

```html
<h1>{{ question.question_text }}</h1>

{% if error_message %}
  <p><strong>{{ error_message }}</strong></p>
{% endif %}

<form action="\{% url 'polls:vote' question.id %\}" method="post">
  {% csrf_token %}
  {% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
  {% endfor %}
  <input type="submit" value="Vote">
</form>
```

**[⬆ back to top](#table-of-contents)**


## Vote View

`/polls/3/vote/` 와 같은 URL이 `detail.html`에서 `submit`버튼을 통해 제출되면 `form`의 `action` attribute에 의해서 POST 방식으로 `vote`로 전달된다.   
`vote` 뷰 함수는 form으로부터 넘어온 `POST` 데이터를 처리한다.

`request.POST['choice']`는 폼에서 키가 `choice`에 해당하는 값인 `choice.id`를 스트링으로 리턴한다. 

`selected_choice.votes`을 1증가시키고, 변경사항을 `selected_choice.save()`을 통해 저장

`HttpResponseRedirect`의 생성자는 인자로 Redirect할 타겟을 인자로 받는데, 이를 위해 `reverse` 함수를 사용해야 한다. `reverse`를 통해 URL을 추출하고 Redirect의 결과로 `results.view`함수를 호출한다.

`POST` 데이터를 처리하는 경우 결과 페이지로 이동하기 위해 `HttpResponseRedirect`객체를 리턴한다

```python
from django.http import HttpResponseRedirect 
from django.urls import reverse

def vote(request, question_id):
  question = get_object_or_404(Question, pk=question_id)
  try:
    selected_choice = question.choice_set.get(pk=request.POST['choice'])
  except (KeyError, Choice.DoesNotExist):
    context = {
      'question': question,
      'error_message': "You didn't select a choice."
    }
    return render(request, 'polls/detail.html', context)
  else:
    selected_choice.votes += 1
    selected_choice.save()
    return HttpResponseRedirect(reverse('polls:results', args=(question.id, )))
```


## Results View

```python
def results(request, question_id):
  question = get_object_or_404(Question, pk=question_id)
  context = {'question': question}
  return render(request, 'polls/results.html', context)
```

## Results Template 
`question` 객체 하나만 있어도 `question 레코드`에 연결된  
`Choice 테이블의 레코드` 모두에 접근할 수 있다. 

```html
<h1>{{ qustion.question_text }}</h1>
<ul>
  {% for choice in question.choice_set.all %}
    <li>
      {{ choice.choice_text }} - {{ choice.votes}} vote{{ choice.votes|pluralize }}
    </li>
  {% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again</a>

<button>
  <a href="/polls/" style="text-decoration: none;">
    Home  
  </a>
</button>
```

**[⬆ back to top](#table-of-contents)**




