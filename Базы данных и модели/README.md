## Базы данных и модели

### Работа с БД

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

