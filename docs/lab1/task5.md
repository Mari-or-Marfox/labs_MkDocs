

## Задание 5
Написать простой веб-сервер для обработки GET и POST HTTP-запросов с помощью библиотеки socket в Python.

Задание:

Сервер должен:
Принять и записать информацию о дисциплине и оценке по дисциплине.
Отдать информацию обо всех оценках по дисциплинам в виде HTML-страницы.
## Код сервера
```python
import socket  # библиотека для работы с сетевыми соединениями
import sys     # библиотека для работы с аргументами командной строки и выходом из программы

# Наш простой HTTP-сервер
class MyHTTPServer:
    def __init__(self, host, port, server_name):
        # Запоминаем адрес и порт, на котором будем работать
        self._host = host
        self._port = port
        self._server_name = server_name
        # Список для хранения оценок студентов
        self._grades = []

    # Главный метод сервера, который будет работать бесконечно
    def serve_forever(self):
        # создаём TCP-сокет (IPv4)
        serv_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # включаем возможность использовать один и тот же адрес/порт сразу после перезапуска сервера
        serv_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        try:
            # привязываем сокет к указанному адресу и порту
            serv_sock.bind((self._host, self._port))
            # начинаем слушать входящие соединения
            serv_sock.listen()
            print(f"{self._server_name} running on {self._host}:{self._port}")

            # бесконечный цикл — сервер работает постоянно
            while True:
                # ждём, пока кто-то подключится
                conn, _ = serv_sock.accept()
                try:
                    # обрабатываем конкретного клиента
                    self.serve_client(conn)
                except Exception as e:
                    # если что-то пошло не так, просто выводим ошибку
                    print('Client serving failed', e)
        finally:
            # при закрытии сервера — закрываем сокет
            serv_sock.close()

    # Метод для обработки одного клиента
    def serve_client(self, conn):
        try:
            # разбираем запрос клиента
            req = self.parse_request(conn)
            # обрабатываем запрос и формируем ответ
            resp = self.handle_request(req)
            # отправляем ответ обратно клиенту
            self.send_response(conn, resp)
        except ConnectionResetError:
            # клиент закрыл соединение внезапно — просто игнорируем
            conn = None
        except Exception as e:
            # если ошибка сервера — отправляем страницу с ошибкой 500
            self.send_error(conn, e)

        # закрываем соединение с клиентом, если оно ещё открыто
        if conn:
            conn.close()

    # Метод для чтения и разбора HTTP-запроса
    def parse_request(self, conn):
        # создаём объект, как файл, чтобы удобно читать построчно
        rfile = conn.makefile('r', encoding='utf-8')
        # читаем первую строку запроса, например: GET / HTTP/1.1
        request_line = rfile.readline().strip().split()
        if len(request_line) != 3:
            # если строка не имеет 3 частей — это невалидный HTTP-запрос
            raise ValueError("Некорректный HTTP-запрос")

        method, url, version = request_line

        # читаем заголовки запроса
        headers = {}
        while True:
            line = rfile.readline().strip()
            if not line:
                break  # пустая строка — конец заголовков
            if ':' in line:
                key, value = line.split(":", 1)
                headers[key.strip()] = value.strip()

        # тело запроса (для POST)
        body = ''
        if method.upper() == 'POST':
            length = int(headers.get('Content-Length', 0))  # длина тела запроса
            body = rfile.read(length)

        # возвращаем всё в виде словаря
        return {
            'method': method.upper(),
            'url': url,
            'version': version,
            'headers': headers,
            'body': body
        }

    # Метод для обработки запроса
    def handle_request(self, req):
        method = req['method']
        url = req['url']
        body = req['body']

        # разбиваем URL на путь и GET-параметры
        if '?' in url:
            path, query_string = url.split('?', 1)
            params = dict(pair.split('=') for pair in query_string.split('&') if '=' in pair)
        else:
            path = url
            params = {}

        if method == 'GET':
            # строим HTML со списком оценок и формой для добавления новой
            html = "<h1>Оценки по дисциплинам</h1><ul>"
            for g in self._grades:
                html += f"<li>{g['discipline']}: {g['mark']}</li>"
            html += "</ul>"
            html += """
            <h2>Добавить оценку</h2>
            <form method="POST">
                Дисциплина: <input name="discipline"><br>
                Оценка: <input name="mark"><br>
                <input type="submit">
            </form>
            """
            return {
                'status': 200,
                'reason': 'OK',
                'body': html
            }

        elif method == 'POST':
            # читаем данные из формы
            post_params = dict(pair.split('=') for pair in body.split('&') if '=' in pair)
            discipline = post_params.get('discipline', 'Не указано')
            mark = post_params.get('mark', 'Не указано')
            # добавляем новую оценку в список
            self._grades.append({'discipline': discipline, 'mark': mark})
            return {
                'status': 200,
                'reason': 'OK',
                'body': "<h1>Данные успешно сохранены</h1><a href='/'>Вернуться</a>"
            }
        else:
            # если метод запроса не GET и не POST — говорим, что нельзя
            return {
                'status': 405,
                'reason': 'Method Not Allowed',
                'body': '<h1>Метод не поддерживается</h1>'
            }

    # Метод отправки ответа клиенту
    def send_response(self, conn, resp):
        status_line = f"HTTP/1.1 {resp['status']} {resp['reason']}\r\n"
        headers = f"Content-Type: text/html; charset=utf-8\r\nContent-Length: {len(resp['body'].encode('utf-8'))}\r\n\r\n"
        conn.sendall(status_line.encode('utf-8'))  # отправляем статус
        conn.sendall(headers.encode('utf-8'))      # отправляем заголовки
        conn.sendall(resp['body'].encode('utf-8')) # отправляем тело HTML

    # Метод для отправки ошибки 500
    def send_error(self, conn, err):
        body = f"<h1>500 Internal Server Error</h1><p>{err}</p>"
        try:
            # формируем стандартный ответ HTTP с ошибкой
            conn.sendall(f"HTTP/1.1 500 Internal Server Error\r\nContent-Length: {len(body.encode('utf-8'))}\r\n\r\n".encode('utf-8'))
            conn.sendall(body.encode('utf-8'))
        except:
            # если даже отправка ошибки не удалась — молча игнорируем
            pass

# точка входа, если файл запускается напрямую
if __name__ == '__main__':
    # читаем аргументы командной строки: адрес, порт и имя сервера
    host = sys.argv[1]
    port = int(sys.argv[2])
    name = sys.argv[3]

    # создаём сервер
    serv = MyHTTPServer(host, port, name)
    try:
        # запускаем сервер навсегда
        serv.serve_forever()
    except KeyboardInterrupt:
        # если Ctrl+C — корректно останавливаем
        print("\nServer stopped")
        sys.exit(0)


```
## Сервер
![Сервер](images/server5.png)
## Html-страница стартовая
![Сервер](images/html1.png)
## Html-страница ввод данных
![Сервер](images/html2.png)
## Html-страница данные сохранены
![Сервер](images/html3_5.png)
## Html-страница отображение данных
![Сервер](images/html4.png)