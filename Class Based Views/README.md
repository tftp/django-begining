## Class Based Views

### класс View

пример использования класса View на примере замены ```def shop_index```:

```
from django.views import View

class ShopIndexView(View):
  def get(self, request: HttpRequest) -> HttpResponse:
    products = Product.object.all()
    context = {
      "products": product
    }
    return render(request, 'shopapp/shop-index.html', context=context)

```

Необходимо подключить его в urls:

```
path("", ShopIndexView.as_view(), name="index")

```

Метод ```as_view()``` превратит класс в функцию.

Пример использования ```post``` запроса на примере ```GroupsListView```:

```
class GroupsListView(View):
  def get(self, request: HttpRequest) -> HttpResponse:
    context = {
      "form": GroupForm(), #описывается в forms.py
      "groups": Group.objects.prefetch_related('permissions').all(),
    }
    return render(request, 'shopapp/groups-list.html', context=context)

  def post(self, request: HttpRequest) -> HttpResponse:
    form  = GroupForm(request.POST)
    if form.is_valid():
      form.save()
    
    return redirect(request.path) #Возвращаем пользователя на туже страницу

```

Также объявляется новая форма ```forms.py```:

```
from django.contrib.auth.models import Group
from django.forms import ModelForm

class GroupForm(ModelForm)
  class Meta:
    model = Group
    fields = ["name"]

```

На странице отображается эта форма в блоке, примерно так:

```
<div>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="sbmit">Create</button>
  </form>
</div>

```

Переход к товару по первичному ключу.

Шаблон ```products-details.html``` :

```
{% block body %}
  <h1>Product <strong>{{product.name }}</strong></h1>
{% endblock %}

```

Функция обработки во ```views.py``` :

```
class ProductDetailsView(View):
  def get(self, request: HttpRequest, pk: int) -> HttpResponse:
    #product = Product.object.get(pk=pk)
    product = get_object_or_404(Product, pk=pk) #Если не найден pk
    context = {
      "product": product,
    }
    return render(request, 'shopapp/products-details.html', context=context)

```

Добавляем динамическую ссылку в urls:

```
path("products/<int:pk>/", ProductDetailView.as_view(), name="product_details")

```

Ссылки на продукты и одиночные продукты по pk будут выглядеть так:

```
<a href="{% url 'shopapp:products_list' %}">Список</a>

<a href="{% url 'shopapp:product_details' pk=product.pk %}">Продукт {{ product.name }} </a>

```

### Template View


