## Базы данных и модели

### 1. Работа с БД

Для работы с БД в pycharm ставится плагин ```database navigator```

Пример подключения к таблице Group через ORM в views.py:

```
#Импортируем модель Group
from django.contrib.auth.models import Group

#Функция использования модели Group
def groups_list(request: HttpRequest):
  context = {
    "groups": Group.objects.prefetch_related('permissions').all(), #метод all() позволяет вытащить все группы
  }
  # prefetch_related делает жадную загрузку связанной модели

  return render(request, 'shopapp/groups-list.html', context=context)

```

В шаблоне ```shopapp/groups-list.html``` выводим список групп:

```
{% if not groups %}
  <h3>No groups yet</h3>
{% else %}
  <ul>
   {% for group in groups %}
     <li>
       <div>{{ group.name }}</div>
       # Если есть связанные данные то их можно вывести:
       <ul>
         {% for permission in group.permissions.all %}
           <li>{{ permission.name }}</li>
         {% endfor %}
       </ul>
     </li>
   {% endfor %}
  </ul>
{% endif %}

```
В urls.py нужно добавить путь по которому откроется список групп:

```
path("groups/", groups_list, name="groups_list")

```

### 2. Модели и поля

Объявляем свои модели в файле models.py:

```
from django.db import models

class Product(models.Model):
  
  #Объявляем поля-свойства для Product
  name = models.Charfield(max_length=100)
  description = models.TextFields(null=False, blank=True)
  price = models.DecimalField(default=0, max_digits=8, decimal_places=2)
  discount = models.SmallIntegerField(default=0)
  created_at = models.DateTimeField(auto_now_add=True)


```

После описания модели нужно запустить миграции для созания таблицы БД

В терминале запускаем создание миграции:

```
python manage.py makemigrations

```

Далее вручную нужно проверить, что миграция правильно создалась в папке ```shopapp/migrations```

Чтобы увидеть какие миграции есть и в каком статусе, выполняется команда:

```
python manage.py showmigrations

```

Чтоы выполнить миграции:

```
python manage.py migrate

```
Чтобы выполнить миграции только для shopapp:

```
python manage.py migrate shopapp

```
Чтобы откатиться на предыдущую миграцию, нужно выполнить команду ```migrate``` и указать номер предыдущей миграции:

```
python manage.py migrate shopapp 0002

```


### Создание django-команд

Создается папка shopapp/management/commands

Внутри этой папки создаются python-файлы, которые представляют из себя отдельные команды

Пример создания команды create_products.py:

```
from django.core.management import BaseCommand
from shopapp.models import Product

class Command(BaseCommand):
  """
  Creates products
  """
  
  def handle(self, *args, **options):
    #тут определяем логику команды
    self.stdout.write("Create products")

      products_name = [
        "Laptop",
        "Desktop",
        "Smartphone"
      ]

      for products_name in products_names:
        product, created = Product.objects.get_or_create(name=products_name)
        self.stdout.write(f"Create product {product.name}")


      self.stdout.write(self.style.SUCCESS("Products create!"))
```

Чтобы увидеть нашу команду в списке доступных команд выполним

```
python mange.py help

```

Чтобы выполнить команду:

```
python manage.py create_products

```
[Документация по моделям](https://docs.djangoproject.com/en/4.1/topics/db/models/)

Для отображения продуктов в views.py:

```
from .models import Product

def products_list(request: HttpRequest):
  context = {
    "products": product.objects.all()
  }
  return render(request, 'shopapp/products-list.html', context=context)

```
При выводе свойств продуктов в шаблоне, интересны следующие команды:

```
<p>Discount: {% firstof product.discount 'no discount' %}</p> #firstof берет первое ненулевое значение

```

### 3. Связи между таблицами

Создаем модель Order в models.py:

```
class Order(models.Model):
  delivery_address = models.TextFields(null=True, blank=True)
  promocode = models.CharField(max_length=20, null=False, blank=True)
  created_at = models.DateTimeFields(auto_now_add=True)

```
Далее создаем миграцию, перед тем как создать связь, чтобы миграции были легко отменяемы.

После создания миграции добавляем строчки создания связи между Order и User:

```
from django.contrib.auth.models import User

...

# в класс Order добавляем ещё одно поле
user = models.ForeignKey(User, on_delete=models.PROTECT)

```
Далее создать и выполнить миграцию с установленной связью.

Пример создания команды создающей заказы, файл shopapp/management/commands/create_order.py

```
#В handle прописываем логику
user = User.objects.get(username="admin") #Задаем пользователя admin
order = Order.objects.get_or_create(
  delivery_address="ul.Pupkin"
  promokode="SALE123"
  user=user
)
self.stdout.write(f"Create order {order}")

```

В views.py прописываем функцию вывода заказов, в которой ссылаемся на связь с пользователем, для жадной подгрузки используется конструкция следующего вида:

```
context = {
  "orders": Order.objects.select_related("user").all(),
}

```
Чтобы добавить продукт в заказ используется связь Многие-ко-многим, для этого в class Order добавляем поле:

```
products = models.ManyToManyFields(Product, related_name="orders")

```
И выполнить миграцию. Django сам создаст нужные таблицы.

Добавление команды обновления заказов и привязка к нему продуктов:

```
#в handle пишем
order = Order.objects.first()
if not order:
  self.stdout.write("no order yet")
  return

products = Product.objects.all()

for product in products:
  order.products.add(product)

order.save()

self.stdout.write(
  self.style.SUCCESS(
    f"Successfully added products {order.products.all()} to order {order}"
  )
)

```
В views.py корректируем жадную загрузку для продуктов:

```
context = {
  "orders": Order.objects.select_related("user").prefetch_related("products").all(),
}

```
[One-to-one relationships](https://docs.djangoproject.com/en/4.0/topics/db/examples/one_to_one/)
[Many-to-one relationships](https://docs.djangoproject.com/en/4.0/topics/db/examples/many_to_one/)
[Many-to-many relationships](https://docs.djangoproject.com/en/4.0/topics/db/examples/many_to_many/)
