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

TemplateView позволяет отрисовывать шаблоны без использования функции render и писать меньше кода, для этого наследуемся от TemplateView, он работает с шаблонами:

```
class ProductsListView(TemplateView):
  template_name = "shopapp/products-list.html"

  def get_context_data(self, **kwargs):
    context = super().get_context_data(**kwargs)
    context["products"] = Product.objects.all()
    return context

```
Если надо описать методы ```get``` или ```post``` то это делается также как и в обычном подклассе View.

 - [Base views](https://docs.djangoproject.com/en/4.1/ref/class-based-views/base/#django.views.generic.base.TemplateView)

### ListView и DetailView

Пример кода для ListView:

```
class ProductsListView(ListView):
  template_name = "shopapp/products-list.html"
  model = Product
  context_object_name = "products"

```
Пример кода для DetailView:

```
class ProductDetailsView(DetailView):
  template_name = "shopapp/products-details.html"
  model = Product
  context_object_name = "product"
  
```
Пример кода ListView и DetailView для Orders:

```
class OrdersListView(ListView):
  #Вместо model указываем queryset, чтобы учесть связи
  queryset = (Order.objects.select_related("user").prefetch_related("product"))

```
Нужно учесть, что используя ListView будут автоматически учитываться шаблон order_list.html и в контексте вместо orders использовать objects_list. Либо описать ```context_object_name = "orders"``` в классе ```OrdersListView```

Пример OrderDetailView:

```
class OrderDetailView(DetailView):
  queryset = (Order.objects.select_related("user").prefetch_related("product"))
  
```
Соответственно шаблон будет называться ```order_detail.html``` в котором:

```
{% block title %}
  Order {{ object.pk }}
{% endblock %}

{% block body %}
  Order {{ object.pk }}

  Ordered by {% firstof object.user.first_name object.user.username %}
  Product in order:
  {% for product in object.products.all %}
    <li>{{ product.name }}</li>
  {% endfor %}

{% endblock %}

```
 -  [Generic display views ](https://docs.djangoproject.com/en/4.1/ref/class-based-views/generic-display/)
 -  [ListView](https://docs.djangoproject.com/en/4.1/ref/class-based-views/generic-display/#listview)
 -  [DetailView](https://docs.djangoproject.com/en/4.1/ref/class-based-views/generic-display/#detailview)

### CreateView и UpdateView

Пример создания CreateView:

```
class ProductCreateView(CreateView):
  model = Product
  fields = "name", "price", "description", "discount"
  success_url = reverse_lazy("shopapp:products_list") #ссылка при успешном создании объекта

```

Для создания нового объекта не нужно создавать формы, они создадуться автоматически, однако если есть такая необходимость то форму можно указать через ```form_class = ProductForm```, в этом случае не используется ```fields```
Теперь нужно правильно создать шаблон и он должен называться ```product_form.html```, его содержимое:

```
...
{% block body %}
  <form method="post">
    {% csrf_token %}
      {{ form.as_p }}
    <button type=submit>Create</button>
{% endblock %}

```

Пример создания UpdateView:

```
class ProductUpdateView(UpdateView):
  model = Product
  fields = "name", "price", "description", "discount"
  
  #Можно также указать success_url или динамически создать ссылку на изменившийся объект через метод
  def get_success_url(self):
    return reverse(
        "shopapp:product_details",
        kwargs={"pk": self.object.pk},
    )

```

Пут в urls будет выглядеть примерно так:

```
path("products/<int:pk>/update/", ProductUpdateView.as_view(), name="product_update")

```
По умолчанию ProductUpdate будет использовать такой же шаблон ```product_form.html```, что и CreateView, чтобы этого избежать можно использовать ```template_name_suffix = "_update_form"``` в ```class ProductUpdateView``` и тогда шаблон для него будет ```product_update_form```

 -  [Generic editing views](https://docs.djangoproject.com/en/4.1/ref/class-based-views/generic-editing/)
 -  [CreateView](https://docs.djangoproject.com/en/4.1/ref/class-based-views/generic-editing/#createview)
 -  [UpdateView](https://docs.djangoproject.com/en/4.1/ref/class-based-views/generic-editing/#updateview)


### Использование DeleteView

Пример использования:

```
class ProductDeleteView(DeleteView):
  model = Product
  success_url = reverse_lazy("shopapp:products_list")

```


