## Базы данных и модели

# Работа с БД

Для работы с БД в pycharm ставится плагин ```database navigator```

Пример подключения к таблице Group через ORM в views.py:

```
#Импортируем модель Group
from django.contrib.auth.models import Group

#Функция использования модели Group
def groups_list(request: HttpRequest):
  context = {
    "groups": Group.objects.all(), #метод all() позволяет вытащить все группы
  }
  return render(request, 'shopapp/groups-list.html', context=context)

```


