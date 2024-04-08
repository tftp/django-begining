## Формы

### Создание форм на основе классов

Создаем файл в приложение forms.py для написания всех форм:

```
from django import forms

class UserBioForm(forms.Form):
  name = forms.Charfield()
  age = forms.IntegerField(label="Your age")
  bio = forms.CharField(label=Biography, widget="forms.Textarea") 
  #label - отображает значение которое будет отображаться перед полем формы
  #widget - меняет отображение поля

```

Чтобы отобразить формы на странице нужно передать экземпляр в контекст, для этого в views.py отредактируем функцию user_form:

```
from .forms import UserBioForms
...

def user_form(request: HttpRequest) -> HttpResponse:
  context = {
    "form": UserBioForm(),
  }
  return render(request, "requestdataapp/user-bio-form.html", context=context)


```

Теперь эту форму указываемв шаблоне user-bio-form.html:

```
...

{% block body %}
  <h1>User form</h1>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}   #указываем отображение формы, можно написать также просто {{ form }}
    <button type="submit">
      Submit
    </button>
  </form>
{% endblock %}

...

```

### Валидация форм

Ограничение по вводу данных в полях формы:

```
name = forms.CharField(max_length=100)
age = forms.IntegerField(min_value=1, max_value=120)

```

Создане формы в shopapp/forms.py:

```
from django import forms

class ProductForm(forms.Form):
  name = forms.Charfield(max_length=100)
  price = forms.DecimakField(min_value=1, max_value=100000)
  description = forms.CharField(label="Product description", widget=forms.Textarea)

```

В папке шаблонов создадним шаблон create-product.html и  включим форму в контекст в views.py:

```
#create-product.html

{% extends 'shopapp/base.html' %}

{% block body %}
  <div>
    <form method="post">
      {% csrf_tocken %}
      
      {{ form.as_p }}

      <button type="submit">Submit</button>
    </form>
  </div>
{% endblock %}


#views.html

from .forms import ProductForm

...

def create_product(request: HttpRequest) -> HttpResponse:
  form = ProducrForm()
  context = {
    "form": form,
  }

  return render(request, 'shopapp/create-product.html', context=context)
  
```

Генерация ссылок:

```
<a href="{% url 'shopapp:product_create' %}"></a>

```

Использование redirect и revese во views.py:

```
from django.shortcuts import render, redirect, reverse

...
if request.method == "POST":
  url = reverse("shopapp:product_list")
  return redirect(url)

```
Пример валидации данных формы на backende в views:

```
def create_product(request: HttpRequest) -> HttpResponse:
  if request.method == "POST":
    form = ProductForm(request.POST) #Теперь есть форма с предзаполненными данными из POST запроса
    if form.is_valid():   #Проверяем валидность

      #name = form.cleaned_data["name"]
      #price = form.cleaned_data["price"]
      #Product.objects.create(name=name, price=price)

      Product.objects.create(**form.cleaned_data)
      url = reverse("shopapp:product_list")
      return redirect(url)
    else:
      form = ProductForm(request.POST)

```

