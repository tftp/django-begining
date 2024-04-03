## Обработка запросов. Middlewares.

### 1. Работа c HTTP-методами

```
#Обращение к методам
{{ request.method }}

#Обращение к ключу запроса name (foo.ru/?name=Jon)
{{ request.GET.name }}

#Или
{% firstof request.GET.name 'User' %}

#Или
a = request.GET.get("a", "")

```

### 2. POST-запросы

Пример:

```
<form method="post">
  {% csrf_token %}
  <label for="name-id">Full name</label>
  <input id="name-id" name="name" type="text" maxlength="100">
  
  <label for="age">Age</label>
  <input id="age" name="age" type="number" min="1" max="99">

  <label for="bio">Bio</label>
  <textarea name="bio" id="bio" cols="42" rows="5"></textarea>

  <button type="submit">
    Submit
  </button>
</form>

{% if request.POST %}
<table>
  <tr>
    <td>{{ request.POST.name }}</td>
  </tr>

  <tr>
    <td>{{ request.POST.age }}</td>
  </tr>

  <tr>
    <td>{{ request.POST.bio|linebreaks }}</td> #фильтр linebreaks чтобы сохранялись переводы строк
  </tr
</table>
{% endif %}

```
Пример формы загрузки файла:

```
#file-upload.html
<form method="post" enctype="multipart/form-data">
  {% csrf_token %}

  <input type="file" name="myfile">

  <button type="submit">Upload</button>
</form>

#views.py
def handle_file_upload(request: HttpRequest) -> HttpResponse:
  if request.method="POST" and request.FILES.get("myfile"):
    myfile = request.FILES["myfile"]
    fs = FileSystemStorage() #Создается экземпляр для сохранения файла
    filename = fs.save(myfile.name, myfile)
    print("save file", filename)


  return render(request, "requestdataapp/file-upload.html")


```
 - [Request and response objects | Django documentation](https://docs.djangoproject.com/en/4.1/ref/request-response/#django.http.HttpRequest.POST)
 - [File storage API | Django documentation](https://docs.djangoproject.com/en/4.1/ref/files/storage/)


