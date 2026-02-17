# Отчет по Первой лабраторной



## Задание 2
Реализовать клиентскую и серверную часть приложения. Клиент запрашивает выполнение математической операции, параметры которой вводятся с клавиатуры. Сервер обрабатывает данные и возвращает результат клиенту.

Вариант:

Теорема Пифагора.

Требования:

Обязательно использовать библиотеку socket.
Реализовать с помощью протокола TCP.
## Код сервера
```python
import socket
import math

# Создаем сокет
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Привязываем сокет к адресу и порту
server_socket.bind(('localhost', 5001))

# Начинаем слушать входящие подключения (ожидание клиентов)
server_socket.listen(1)
print("Сервер запущен на порту 5001...")

while True:
    # Принимаем соединение от клиента
    client_connection, client_address = server_socket.accept()
    print(f'Подключение от {client_address}')

    # Получаем сообщение от клиента
    request = client_connection.recv(1024).decode().strip()
    print(f'Запрос от клиента: {request}')

    try:
      a, b = request.split()
      a = float(a)
      b = float(b)
      c = math.sqrt(a**2+b**2)
      response = f"Гипотенуза: {c}"
    except Exception as e:
      response = f"Ошибка: {e}"
    # Отправляем ответ клиенту
    client_connection.sendall(response.encode())

    # Закрываем соединение
    client_connection.close()
```
## Код клиента
```python
import socket

# Создаем сокет
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Подключаемся к серверу
client_socket.connect(('localhost', 5001))

# Отправляем сообщение серверу
message = input("введите два катета через пробел: ")
client_socket.sendall(message.encode('utf-8'))

# Получаем ответ от сервера
response = client_socket.recv(1024)
print(f'Ответ от сервера: {response.decode()}')

# Закрываем соединение
client_socket.close()
```
## Сервер
![Сервер](images/server12.png)
## Клиент
![Сервер](images/client12.png)

