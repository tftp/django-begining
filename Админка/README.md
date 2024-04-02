## Админка

### 1. Подключение моделей к админке

Для подсоединения модели к админке, в файле admin.py импортируем модель:

```
#Добавляем строки для отображения модели в админке

from .models import Product

admin.site.register(Product)

```
Если мы хотим видеть определенные поля модели, то нужно добавить строки:

```
class ProductAdmin(admin.ModelAdmin):
  list_display = "pk", "name", "description", "price", "discount"

#После этого нужно эту модель подключить к админке

admin.site.register(Product, ProductAdmin)

```

В админке можно видеть модель и её объекты а также переходить на объект модели для редактирования. По умолчанию для перехода на объект нужно нажать на "pk", но если требуется назначить другое поле, например, "name":

```
#Для этого в классе ProductAdmin объявляем ещё одно поле
list_display_links = "pk", "name"

```
Вместо команды ```admin.site.register``` можно использовать декоратор ```@admin.register```:

```
@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
  list_display = "pk", "name", "description", "price", "discount"
  list_display_links = "pk", "name"

```
Если нужно изменить представление объекта в заголовке, при редактировании в админке, то в models.py добавим в класс Product

```
def __str__(self) -> str:
  return f"Product(pk={self.pk}, name={self.name!r})"

```

Отображение полей в админке, по примеру ```description```, если нужно ограничить длину отображаемого текста, то в models.py в класс Product добавляем ещё поле:

```
@property
def description_short(self) -> str:
  if len(self.description) < 48:
    return self.description
  return self.description[:48] + "..."

```
Теперь поле ```description_short``` нужно указать в админке admin.py:

```
list_display = "pk", "name", "description_short", "price", "discount"

```
Если данныое поле используется только в админке, его описание можно перенести из models.py в admin.py:

```
def description_short(self, obj: Product) -> str:
  if len(obj.description) < 48:
    return obj.description
  return obj.description[:48] + "..."

```

### 2. Фильтры и поле поиска

Добавляем в admin.py в class ProductAdmin, для сортировки по полю "pk":

```
ordering = "pk",

```

Добавление поиска по записям, добавляем в класс ProductAdmin:

```
search_filds = "name", "description"

```

### 3. Отображение и редактирование связанных записей

Добавим модель Order в admin.py:

```
@admin.register(Order)
class OrderAdmin(admin.ModelAdmin):
  list_display = "delivery_address", promocode", created_at", "user"

  #чтобы использовать жадную загрузку user создаем метод 
  def get_queryset(self, request):
    return Order.objects.select_related("user")

  #Для отображения того или иного поля user создаем метод
  def user_verbose(self, obj: Order) -> str:
    return obj.user.first_name or obj.user.username
  
  # Если есть user_verbose то добавляем его в list_display
  # list_display = "delivery_address", promocode", created_at", "user_verbose"

```
Для подключения связаных записей, их редактирования в админ панели, нужно подключить ещё одну модель в admin.py:

```
class ProductInline(admin.TabularInline):
 model = Order.products.through

```
Теперь указываем эту связь в OrderAdmin:

```
#Добавляем в OrderAdmin
inlines = [ 
  ProductInline,
]

```
Хорошей практикой будет указать жадную подгрузку products в order:

```
  def get_queryset(self, request):
    return Order.objects.select_related("user").prefetch_related("products")

```

Вместо ```class ProductInline(admin.TabularInline)``` можно написать ```class ProductInline(admin.StackedInline)```, изменится только ототбражение связанных записей, но функции останутся те же.


[The Django admin site](https://docs.djangoproject.com/en/4.1/ref/contrib/admin/#inlinemodeladmin-objects)

### 4. Группировка полей

Для решения этой задачи используется fieldsets = [ИмяСекции, {Параметры}], добавим в ProductAdmin:

```
fieldsets = [
  (None, {
    "fields": ("name", "description"),
  }),

  ("Price options", {
    "fields": ("price", "discount"),   #Поля в секции
    "classes": ("collapse", "wide"),   #collapse - Добавляет возможность Свернут/Развернуть секцию, wide - Добавляет отступ
  })
  
  ("Extra options", {
    "fields": ("archived",),
    "classes": ("collapse",),
    "descriptions": "Extra options. Field 'archived' is for soft delete",
  })
]

```

### 5. Групповые действия

В админке можно удалять записи, но как сделать, чтобы их не удалять а помечать, как удаленные?
Для этого нужно создать определенные действия.
Добавим поле ```archived``` в ProductAdmin:

```
list_display = "pk", "name", "description_short", "price", "discount", "archived"

```
Далее создаем действие которое будет архивировать записи с новой функцией merk_archived:

```
@admin.action(description="Archive product")  #Дескриптор создает в админке действие
def merk_archived(modeladmin: admin.ModelAdmin, request: HttpRequest, queryset: QuerySet):
  queryset.update(archived=True)

```

Чтобы указать, что это действие относится к ProductAdmin, внутри класса добавляем:

```
actions = [
  mark_archived,
]

```

Теперь для Products доступен новый Action для архивирования продуктов.

По аналогии делается Действие для разархивирования продуктов.

Миксины (Примеси) - это отдельные классы, которые реализуют одно действие.

Для миксинов создадим новый файл ```admin_mixins.py``` в котором будет вся логика

```
from django.db.models import QuerySet
from django.http import HttpRequest, HttpResponse
from django.db.models.options import Options

class ExportAsCSVMixin:
  def export_csv(self, request: HttpRequest, queryset: QuerySet);
    #Здесь реализуем логику действия
    meta: Options = self.model._meta #meta возьмет список всех полей модели
    field_names = [field.name for field in meta.fields]

    response = HttpResponse(content_type="text/csv")
    response["Content-Disposition"] = f"attachment; filename={meta}-export.csv"

    csv_writer = csv.writer(response)

    csv_writer.writerow(field_names)

    for obj in queryset:
      csv_writer.writerow([getattr(obj, field) for field in field_name])

    return response

  export_csv.short_description = "Export as CSV"


```

Теперь в ```admin.py``` экспортируем миксин

```
from .admin_mixins import ExportAsCSVMixin
...
#Подмешаем в ProductAdmin
class ProductAdmin(admin.ModelAdmin, ExportAsCSVMixin):
  #добавим в actions действие export_csv
  actions = [
    mark_archived,
    mark_unarchived,
    "export_csv",
  ]
...

```


