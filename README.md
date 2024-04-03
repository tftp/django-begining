# Django for beginners

## Введение

### 1. Запуск Django-проекта

```
pip install django

pip freeze > requirements.txt

python -m django startproject mysite

cd mysite
```
Запуск сервера проекта

```
python manage.py runserver
```
После запуска сервера, имеем доступ к нему через ```http://127.0.0.1:8000```

В файле проекта ```urls.py``` есть переменная ```urlpatterns```, где прописываем доступные пути проекта. При инициализации проекта описан только один путь для доступа к административной панели.

Проводим первую миграцию (создание базы данных для создания таблицы пользователей) и создаем суперпользователя:
```
python manage.py migrate

python manage.py createsuperuser
```

Настройки базы данных прописаны в ```settings.py```.
Для комфортной работы с БД в Pycharm можем поставить плагин Database Navigator.

Для знакомства с командами manage.py используем команду ```python manage.py help```

### 2. Создание Django-приложения

Для создания django-приложения shopapp:

```
python manage.py startapp shopapp
```
В django-проекте появится папка shopapp с содержимым. В apps.py описывается настрока приложения. Для включения его в проект, копируем название класса приложения (ShopappConfig) и включаем его в settings.py в переменную INSTALLED_APPS, как ```shopapp.apps.ShopappConfig```.

Создаем новый файл в приложении shopapp для роута: ```shopapp/urls.py```, в нем прописываем:

```
from django.urls import path
from .views import shop_index

app_name = "shopapp"

urlpatterns = [
   path("", shop_index, name="index"),
]
```

А в файл mysite/urls.py в переменную urlpatterns добавляем ```path('shop/', include('shopapp.urls'))```

В файл shopapp/views.py записываем представление:


```
from django.http import HttpResponse, HttpRequest
from django.shortcuts import render

def shop_index(request: HttpRequest):
  context = {
    "time_running": default_timer(),
  }
  return render(request, 'shopapp/shop-index.html', context=context)

```

### 3. Django-шаблоны

Для использования шаблонов создаем папку shopapp/templates/shopapp в которой создаются шаблоны html, например shop-index.html, который мы используем в shopapp/views.py

Для вывода переменных в html шаблонах используется вставки ```{{ time_running }}``` и запуск встроенных функций ```{% lorem 3 p random %}```

Обычно создается базовый шаблон, например base.html в котором используются вставки из блоков:

```
<head>
  <title>
    {% block title %}
      Base Title
    {% endblock %}
  </title>
</head>
<body>
  {% block body %}
    Base body
  {% endblock %}
  <div>
   {% now 'H:i' %}
  </div>

  <div>
   {% now 'l' as current_weekday %}
   Today is {{ current_weekday }}
  </div>
</body>
```
И второстепенный шаблон shop-index.html будет выглядеть так:

```
{% extends 'shopapp/base.html' %}

{% block title %}
  Shop Index
{% endblock %}

{% block body %}
<div>
  Time running: {{ time_running }}
</div>

<div>
  {% lorem %}
</div>

<% endblock %}
```

Если мы передаем в контексте список продуктов в переменной products, то используем конструкцию for для вывода списка продуктов:

```
{% for product in products %}
  <li>{{ product }}</li>
{% endfor %}
```
или

```
{% for name, price in products %}
  <li>{{ name }} for {{ price }}</li>
  {% if name|length_is:'7' %}
    <span>Lucky product</span>
  {% endif %}
{% empty %}
  No products here
{% endfor %}
```

