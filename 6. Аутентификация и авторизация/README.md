## Аутентификация и авторизация

### View для аутентификации

Для примера создается приложение ```myauth```

Создадим ```myauth/templates/myauth/login.html```

```
{% extends base.html %}

{% block body %}

  <form method="post">
    {% csrf_token %}

    {% if error %}
      {{ error }}
    {% endif %}

    <label for="username">Username:</label>
    <input type="text" id="username" name="username" required>

    <label for="password">Password:</label>
    <input type="text" id="password" name="password" required>

    <button type="submit">Login</button>
  </form>

{% endblock %}

```
Создание View:

```
from django.contrib.auth import authenticate, login

def login_view(request: HttpRequest):
  if request.method == "GET":
    if request.user.is_authenticated:
      return redirect('/admin/') #Если уже прошел аутентификацию, то перенаправляем куда нибудь
    
    return render(request, 'myauth/login.html')

  username = request.POST["username"]
  password = request.POST["password"]

  user = authenticate(request, username=username, password=password)
  if user is not None:
    login(request, user)
    return redirect("/admin/")

  return render(request, "myauth/login.html", {"error": "invalid login credentials"})


```
В urls создаем путь:

```
path("login/", login_view, name="login")

```

### Стандартные view для аутентификации
Для подключения стандартной страницы аутентификации, достаточно подключить её через LoginView в urls:

```
from django.contrib.auth import views

path(
  "login/", 
   views.LoginViews.as_view(
     template_name="myauth/login.html",  #переназначаем шаблон по умолчанию
     redirect_authenticated_user=True,   #нужно для перенаправления аутентифицированных пользователей
   ), 
   name="login")

```

Если используется стандартная view то в шаблоне нужно переделать вывод формы в ```login.html```  через:

```
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
  </form>

```
При удачной авторизации происходит перенаправление на страницу профиля, перенаравление можно переназначить в ```settings.py```, введя переменную:

```
LOGIN_REDIRECT_URL = '/admin/'

```

 - [Authenticating users](https://docs.djangoproject.com/en/4.1/topics/auth/default/#authenticating-users)
 - [How to log a user in](https://docs.djangoproject.com/en/4.1/topics/auth/default/#how-to-log-a-user-in)
 - [Using the Django authentication system](https://docs.djangoproject.com/en/4.1/topics/auth/default/#django.contrib.auth.views.LoginView)

### Сессии и куки

Пример использования куки:

```
def set_cookie_view(request: HttpRequest) -> HttpResponse:
  response = HttpResponse("Cookie set")
  response.set("variable", "value", max_age=3600)
  return response


def get_cookie_view(request: HttpRequest) -> HttpResponse:
  value = request.COOKIES.get("variable", "default_value")
  return HttpResponse(f"Cookie value: {value!r}")

```
Пример использования сессий:

```
def set_session_view(request: HttpRequest) -> HttpResponse:
  response = HttpResponse("Session set")
  request.session["variable"] =  "value"
  return response

def get_session_view(request: HttpRequest) -> HttpResponse:
  value = request.session.get("variable", "default_value")
  return HttpResponse(f"Session value: {value!r}")

```
 - [How to use sessions | Django documentation](https://docs.djangoproject.com/en/4.1/topics/http/sessions/)

### Использование logout

```
def logout_view(request: HttpRequest):
  logout(request)
  return redirect(reverse("myauth:login"))

#Или использовать класс

class MyLogoutView(LogoutView):
  next_page = reverse_lazy("myauth:login")

#Или использовать функцию logout_then_login в urls:
  path("logout/",views.logout_then_login, name="logout")

#для редиректа logout_then_login нужно в settings добавить переменную:
LOGIN_URL = '/login/'


```

