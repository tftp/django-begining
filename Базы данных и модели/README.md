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

### 2. Модели и поля

Объявляем свои модели в файле models.py:

```
from django.db import models

class Product(models.Model):
  
  #Объявляем поля-свойства для Product
  name = models.Charfield(max_length=100)
  description = models.TextFields(null=False, blank=True)

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

