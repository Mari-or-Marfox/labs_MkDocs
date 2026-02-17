# Отчет по Первой лабраторной

## Задание 1

Реализовать клиентскую и серверную часть приложения. Клиент отправляет серверу сообщение «Hello, server», и оно должно отобразиться на стороне сервера. В ответ сервер отправляет клиенту сообщение «Hello, client», которое должно отобразиться у клиента.

Требования:

Обязательно использовать библиотеку socket.
Реализовать с помощью протокола UDP.
## Код сервера
```python
import socket
#Создаем сокет
server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

Привязываем сокет к адресу и порту

server_socket.bind(('localhost', 8000))

while True:

    data, client_address = server_socket.recvfrom(1024)

    print(f'Подключение от {client_address}, Запрос от клиента: {data.decode("utf-8")}')

#Отправляем ответ клиенту

    response = 'Привет от сервера!'

    server_socket.sendto(response.encode('utf-8'), client_address)
```
## Код клиента
```python
import socket

#Создаем сокет

client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

#Отправляем сообщение серверу

client_socket.sendto('Привет, сервер!'.encode('utf-8'), ('localhost', 8000))

#Получаем ответ от сервера

response, server_address = client_socket.recvfrom(1024)

print(f'Ответ от сервера: {response.decode("utf-8")}')

client_socket.close()
```
## Сервер
![Сервер](images/server.png)
## Клиент
![Сервер](images/client.png)


