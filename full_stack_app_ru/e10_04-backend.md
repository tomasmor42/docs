## Бэкенд проекта

Рассмотрим как подобное разбиение может быть осуществлено на практике. Мы будем разрабатывать приложение для оценки напитков. Пользователь может записать где и когда он что-то пил и какое впечатление от напитка осталось &mdash; в базовом функционале понадобится модель для напитков и какой-то числовой рейтинг у них. Начнём разработку с бэкенда, но сначала создадим две папки (по одной под каждую часть проекта):

```
drinkrate/
├── backend
└── frontend
```

По нашей задумке backend и frontend хранятся в разных репозиториях. В конце модуля мы приведём ссылки на репозитории с кодом проекта, который сейчас будем разрабатывать.

Переходим в папку с бэкендом, создаём виртуальную среду, активируем её, ставим туда django и django-rest-framework:

```
cd drinkrate/backend
python3.7 -m venv venv
source venv/bin/activate
pip install django django-rest-framework
```

Так установится django последней версии. Дальнейший текст модуля приводится на примере `django==3.0.5`. Создаём проект и первое приложение в нём:

```
django-admin startproject drinkrate
cd drinkrate
./manage.py startapp drinks
```

К этому моменту код нашего бэкенда выглядит так:

```
backend/
├── venv
└── drinkrate
    ├── drinkrate
    │   ├── asgi.py
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── drinks
    │   ├── admin.py
    │   ├── apps.py
    │   ├── __init__.py
    │   ├── migrations
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── tests.py
    │   └── views.py
    └── manage.py
```

Если у вас получилось после повторения команды выше что-то другое, то убедитесь что вы выполняли команды из всех тех же папок, не пропуская переходы.

Создадим модель для нашего напитка

```python
# backend/drinkrate/drinks/models.py
from django.db import models


class Drink(models.Model):
    name = models.CharField('название', max_length=128)
    description = models.TextField('описание', null=True)
    rating = models.IntegerField()
```

Приложение надо будет добавить в `INSTALLED_APPS`, заодно включим уже установленный `django-rest-framework`. Релевантная часть файла `backend/drinkrate/drinkrate/settings.py`:

```python
# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'drinks',
]
```
В этом месте пора остановиться и запустить девелоперский сервер, чтобы убедиться что все нужные приложения подключены и добавлены правильно:

```
./manage.py makemigrations drinks
./manage.py migrate
./manage.py runserver
```

По адресу http://localhost:8000 нас должна поприветствовать стандартная заглушка Django:

![django-start](img/e10-04-1-django-start.png)

Применение миграции автоматически создаст базу sqlite, потому что такие настройки у проекта по умолчанию. Так как мы планируем использовать DRF, нам нужно добавить сериализаторы и нужные пути для работы с нашей пока что одинокой моделью. Сериализатор и viewset можно хранить в одном файле внутри приложения `drinks`. Туда же добавим функцию для создания роутера:

```python
# bakend/drinkrate/drinks/api_views.py
from rest_framework import routers, viewsets, serializers
from .models import Drink

class DrinkSerializer(serializers.ModelSerializer):
    class Meta:
        model = Drink
        fields = ['id', 'name', 'description', 'rating']

class DrinksViewSet(viewsets.ModelViewSet):
    queryset = Drink.objects.all()
    serializer_class = DrinkSerializer

def get_router_views():
    router = routers.DefaultRouter()
    router.register('drinks', DrinksViewSet)
    return router.urls
```

Роутер для модели надо добавить в головной `urls.py`, он расположен в папке `drinkrate/drinkrate`:

```python
# backend/drinkrate/drinkrate/urls.py
from django.contrib import admin
from django.urls import path, include

from drinks.api_views import get_router_urls

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include(get_router_urls())),
]
```

После этого, во-первых, в браузере по адресу `http://localhost:8000` должно показывать 404, а во-вторых, по адресу `http://localhost:8000/api/v1/drinks/` мы должны увидеть страницу с пустым списком напитков:

![empty drf](img/e10-04-2-drf-empty.png)

Чтобы они появились, нам потребуется определить модель для админки:

```python
# backend/drinkrate/drinks/admin.py
from django.contrib import admin
from .models import Drink

@admin.register(Drink)
class DrinkAdmin(admin.ModelAdmin):
    fields = ['name', 'description', 'rating']
```

Если после этого создать суперюзера (`./manage.py createsuperuser`), то можно будет создать новые объекты. Но юзера можно не создавать и воспользоваться API или вообще набросать свой файл с фикстурой и загрузить его командой `./manage.py loaddata`. Вот пример такого файла:

```json
[
  {
    "model": "drinks.drink",
    "pk": 1,
    "fields": {
      "name": "кофе",
      "description": "бодрящий напиток, слегка кислит",
      "rating": 10
    }
  },
  {
    "model": "drinks.drink",
    "pk": 2,
    "fields": {
      "name": "чай",
      "description": "лучшее что есть после кофе",
      "rating": 9
    }
  },
  {
    "model": "drinks.drink",
    "pk": 3,
    "fields": {
      "name": "берёзовый сок с мякотью",
      "description": "скрипит на зубах",
      "rating": 5
    }
  }
]
```
Загрузить в админку его можно с помощью `./manage.py loaddata drinks.json`. После этого на странице API будут новые объекты:

![drf some drinks](img/e10-04-3-drf-some-drinks.png)

Напоминаем также, что в запрос браузеру можно добавлять GET-параметр, указывающий тип ответа `http://localhost:8000/api/v1/drinks/?format=json`, а также можно использовать `httpie` для доступа к API:

```
 $ http :8000/api/v1/drinks/
HTTP/1.1 200 OK
Allow: GET, POST, HEAD, OPTIONS
Content-Length: 336
Content-Type: application/json
Date: Wed, 02 Apr 2020 17:31:00 GMT
Server: WSGIServer/0.2 CPython/3.7.6
Vary: Accept, Cookie
X-Content-Type-Options: nosniff
X-Frame-Options: DENY

[
    {
        "description": "бодрящий напиток, слегка кислит",
        "id": 1,
        "name": "кофе",
        "rating": 10
    },
    {
        "description": "лучшее что есть после кофе",
        "id": 2,
        "name": "чай",
        "rating": 9
    },
    {
        "description": "скрипит на зубах",
        "id": 3,
        "name": "берёзовый сок с мякотью",
        "rating": 5
    }
]
```

Итак, у нас готов наш бэкенд в виде API, нет никаких пользователей и мы можем переключиться на фронтенд.