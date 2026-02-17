# Отчет по Первой лабраторной

## Реализация первого приложения.

## Практическое задание 1
Необходимо установить Django Web framework любым доступным способом.

Формат именований файлов:
Формат имени Django-проекта: “django_project_фамилия”.
Формат имени Django-приложения: “project_first_app”.
## Решение
![Сервер](images/Django.png)
## Практическое задание 2.1
В проекте создать модель данных об автовладельцах 
## Решение
```python
from django.db import models


class Owner(models.Model):
    last_name = models.CharField(max_length = 30)
    first_name = models.CharField(max_length = 30)
    birth_date = models.DateTimeField(null = True, blank = True)

    def __str__(self):
        return f"{self.last_name} {self.first_name}"


class Car(models.Model):
    plate_number = models.CharField(max_length = 15)
    brand = models.CharField(max_length = 20)
    model = models.CharField(max_length = 20)
    color = models.CharField(max_length = 30, null = True, blank = True)

    def __str__(self):
        return f"{self.brand} {self.model} ({self.plate_number})"


class Ownership(models.Model):
    owner = models.ForeignKey(Owner, on_delete = models.CASCADE, related_name = "ownerships")
    car = models.ForeignKey(Car, on_delete = models.CASCADE, related_name = "ownerships")
    start_date = models.DateTimeField()
    end_date = models.DateTimeField(null = True, blank = True)

    def __str__(self):
        return f"{self.owner} - {self.car}"


class DriverLicense(models.Model):
    owner = models.ForeignKey(Owner, on_delete = models.CASCADE, related_name = "licenses")
    license_number = models.CharField(max_length = 10)
    category = models.CharField(max_length = 10)
    issue_date = models.DateTimeField()

    def __str__(self):
        return f"{self.license_number} ({self.owner})"

```

## Практическое задание 3
Необходимо заполнить таблицы данными средствами админ-панели.
Зарегистрировать владельца авто в админ-панели. Админ-панель позволяет работать с объектами базы данных (операции создания, удаления, редактирования). Необходимо зайти в файл admin.py в папке приложения (*_app) и зарегистрировать владельца автомобиля следующими командами:
```python
from .models import Название владельца в модели данных 
#то, что указано в кавычках - это какие либо параметры, которые Вам нужно ввести самостоятельно, в соответствии с Вашим проектом. В Конкретном случае, Вам нужно указать название таблицы модели данных, которую необходимо отразить в админ-панеле.
admin.site.register(Название владельца в модели данных)
```
Зарегистрировать остальные таблицы модели данных в админ-панели.
Создать суперпользователя командой:
```python
python manage.py createsuperuser
```
Запустить сервер командой:
```python
python manage.py runserver
```
Зайти в админ-панель по url-адресу (127.0.0.1:8000/admin/) и добавить двух владельцев автомобилей, 4 автомобиля. Далее связать каждого владельца минимум с тремя автомобилями, так, чтобы не было пересечений по датам владения и продажи.
## Практическое задание 4
Создать в файле views.py (находится в папке приложения) представление (контроллер), который выводит из базы данных данные о владельце автомобиля. 
Создать страницу html-шаблона owner.html в папке templates (создать папку templates в корне проекта, если ее нигде нет, далее в контекстном меню папки создать html-файл). Страница должна содержать отображение полей переданных из контроллера.
## Практическое задание 5
Создать файл адресов urls.py в папке приложения (*_app) (пока пустой).
Импортировать файл urls.py приложения в проект (модифицировать файл urls.py в той папке, в которой хранится файл setting.py). Файл должен иметь подобный код:
```python
from django.contrib import admin
from django.urls import path, include
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('example6_app.urls')), # данная строчка импортирует в проект отдельный файл юрлов Вашего приложения (example6_app - название Вашего приложения (название папки)), urls указывает на файл юрлов в папке приложения (указывает на тот пустой файл, который мы создали в пункте 9.а.
]
```
Теперь необходимо описать в файле urls.py приложения, созданном в пункте 4.1.1, такой url-адрес, который сможет обратиться к контроллеру и вывести страницу, которая должна быть отрендерена контроллером.
Необходимо обратиться к контроллеру, который создан в задаче 4 и передать параметр (иденфикационный номер владельца) в адресной строке

## Сервер
![Сервер](images/server.png)
## Клиент
![Сервер](images/client.png)


