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

Использование redirect и reverse во views.py:

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
      form = ProductForm()
...

```
Добавление валидатора:

```
from django.core import validators
...

class ProductForm(forms.Form):
  name = forms.CharField(max_length=100)
  price = forms.DecimalFiels(min_value=1, max_value=100000, decimal_places=2)
  description = forms.CharField(
    label="Product description",
    widget=forms.Textarea(attrs={"rows": 5, "cols": 30}),
    validators=[validators.RegexValidator(    #Используем встроенный валидатор RegexValidator
      regex=r"greate",
      message="Field must contain word 'great'",
    )],
  )

```
Форма для загрузки файлов:

```
class UploadFileForm(forms.Form):
  file = forms.FileField()

```

тогда во views.py сошлемся на эту форму:

```
...
def handle_file_upload(request: HttpRequest) -> HttpResponse:
  if request.method="POST" and request.FILES.get("myfile"):
    form = UploadFileForm(request.POST, request.FILES)
    if form.is_valid():
      #myfile = request.FILES["myfile"]
      myfile = form.cleaned_data["file"]
      fs = FileSystemStorage() #Создается экземпляр для сохранения файла
      filename = fs.save(myfile.name, myfile)
      print("save file", filename)
  else:
   form = UploadFileForm()
 
  context = {
    "form": form,
  }
  ...

```

Создание функций валидаторов.

```
#forms.py

from django.core.files.uploadfile import InMemoryUploadFile
from django.core.exceptions import ValidationError

def validate_file_name(file: InMemoryUploadFile) -> None:
  if file.name and "virus" in file.name:
    raise ValidationError("file name should not contain 'virus'")

#подключаем нашу функцию как валидатор
class UploadFileForm(forms.Form):
  file = forms.FileField(validator=[validate_file_name])

```

### Model Form

Позволяет сгенерировать форму на основе модели.

Реализация моделей через формы в forms.py:

```
from .models import Product

class ProductForm(forms.ModelForm):
  class Meta:
    model = Product
    fields = "name", "price", "description", "discount"

```

Так как ModelForm связан с моделью, то во views.py можно не использовать строчку

```
      Product.objects.create(**form.cleaned_data)

```

а вместо неё писать строчку:

```
    form.save()

```


 - [Form and field validation | Django documentation](https://docs.djangoproject.com/en/4.1/ref/forms/validation/)
 - [Creating forms from models](https://docs.djangoproject.com/en/4.1/topics/forms/modelforms/)

