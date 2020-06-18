# hello world django

django로 만들어 보는 hello world~
본격적으로 django를 학습하기에 앞서서 django의 장점에 대해 알아봅시다

## django 

### django란 무엇인가?
djagno는 웹 프로그램을 개발하는 데 사용하는 파이썬 웹 프레임워크입니다. 제공하는 기능이 풍부하여 쉽고 빠르게 웹 개발이 가능합니다. 

### django의 특징

- MVC 패턴 기반 MVT
- 객체 관계 매핑
- 자동으로 구성되는 관리자 화면
- 우아한 URL 설계
- 자체 템플릿 시스템
- 캐시 시스템
- 다국어 지원
- 풍부한 개발 환경
- 소스 변경사항 자동 반영


## django의 애플리케이션 개발 방식

웹 사이트를 만들 때 가장 먼저 해야 할 작업은 프로그램이 해야 할 일을 적당히 나누어서 모듈화하는 일입니다. 
장고에서는 용어를 사용할 때 웹 사이트에 대한 전체 프로그램을 project라고 하고, 모듈화된 단위 프로그램을 application이라고 부릅니다. 즉, 애플리케이션이 모여서 프로젝트가 되는 개념입니다. 

### django가 가지는 최강의 이점
모든 애플리케이션 개발에 반드시 필요한 파일들은 django가 알아서 생성해주므로 개발자는 그 내용을 채워넣기만 하면 됩니다. 즉, **개발자들이 어떤 파일들을 만들어야 할지 고민할 필요가 없습니다.** 


### MVT 패턴 
이 부분은 책을 보며 함께 학습니다. 


## django의 시작


### 프로젝트 뼈대를 만들기 위한 명령어

| 명령어                           | 역할                                    |
| -------------------------------- | --------------------------------------- |
| django-admin startproject mysite | mysite라는 프로젝트를 생성              |
| python manage.py startapp polls  | polls 라는 애플리케이션을 생성          |
| python manage.py makemigrations  | 앱의 모델 변경 사항을 정리              |
| python manage.py migrate         | 설정 파일을 확인 및 수정함              |
| python manage.py runserver       | 현재까지 작업을 개발용 웹 서버로 확인함 |
| python manage.py createsuperuser | 관리자 계정 생성                        |



## hello world 프로젝트 시작

- hello world를 웹 사이트에 표현해보는 간단한 프로젝트를 시작합니다.
- windows 환경일 경우는 아래 명령어를 console에 그대로 입력하면 되지만 맥의 경우는 조금 다릅니다.
- [맥의 경우는 설정 방법이 조금 다를 수 있습니다.](https://freehoon.tistory.com/135)


### django setting
프로젝트를 진행할 폴더 생성

```
mkdir helloworld
cd helloworld
```

장고 설치

```python
pip install django
```

### config setting (프로젝트 생성)
아래 명령어를 프로젝트 전체의 설정을 담당하는 폴더를 생성합니다. 

```python
django-admin startproject config . # O
django-admin startproject config  # X
```

### migrate

```python
python manage.py migrate
```



### 작업 내역 확인을 위해 서버 실행

```python
python manage.py runserver
```

### 관리자 계정 생성

```python
python manage.py createsuperuser

# 계정: admin
# 이메일: Your email
# 비밀번호: admin1234
```

- 관리자 계정 생성 후 
- http://127.0.0.1:8000/admin/ 에서 관리자 접속



--- 

### 애플리케이션 생성

```python
python manage.py startapp polls
```