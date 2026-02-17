

## Задание 3
Реализовать серверную часть приложения. Клиент подключается к серверу, и в ответ получает HTTP-сообщение, содержащее HTML-страницу, которая сервер подгружает из файла index.html.

Требования:

Обязательно использовать библиотеку socket.
## Код сервера
```python
import socket

HOST = 'localhost'
PORT = 5002

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind((HOST, PORT))
server_socket.listen(5)

print(f"HTTP сервер запущен на {HOST}:{PORT}...")

while True:
    client_connection, client_address = server_socket.accept()
    print(f'Подключение от {client_address}')

    request = client_connection.recv(1024).decode('utf-8')
    print(f'Запрос клиента:\n{request}')

    try:
        # Открываем и читаем HTML-файл
        with open("index.html", "r", encoding="utf-8") as file:
            html_content = file.read()

        body = html_content.encode('utf-8')  # преобразуем в байты

        # Формируем HTTP-ответ
        response = (
            "HTTP/1.1 200 OK\r\n"
            "Content-Type: text/html; charset=UTF-8\r\n"
            f"Content-Length: {len(body)}\r\n"
            "Connection: close\r\n"
            "\r\n"
        ).encode('utf-8') + body

    except FileNotFoundError:
        body = "Файл index.html не найден".encode('utf-8')
        response = (
            "HTTP/1.1 404 Not Found\r\n"
            "Content-Type: text/plain; charset=UTF-8\r\n"
            f"Content-Length: {len(body)}\r\n"
            "\r\n"
        ).encode('utf-8') + body

    # Отправляем ответ клиенту
    client_connection.sendall(response)
    client_connection.close()
```
## Сервер
![Сервер](images/server3.png)
## Html-код
![Сервер](images/html.png)
## Html-страница
![Сервер](images/html3.png)
