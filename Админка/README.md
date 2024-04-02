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


