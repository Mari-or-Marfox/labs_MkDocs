

## Задание 5
Написать простой веб-сервер для обработки GET и POST HTTP-запросов с помощью библиотеки socket в Python.

Задание:

Сервер должен:
Принять и записать информацию о дисциплине и оценке по дисциплине.
Отдать информацию обо всех оценках по дисциплинам в виде HTML-страницы.
## Код сервера
```python
import socket
import sys

class MyHTTPServer:
    def __init__(self, host, port, server_name):
        self._host = host
        self._port = port
        self._server_name = server_name
        self._grades = []  # Хранилище оценок

    def serve_forever(self):
        serv_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM, proto=0)
        serv_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        try:
            serv_sock.bind((self._host, self._port))
            serv_sock.listen()
            print(f"{self._server_name} running on {self._host}:{self._port}")

            while True:
                conn, _ = serv_sock.accept()
                try:
                    self.serve_client(conn)
                except Exception as e:
                    print('Client serving failed', e)
        finally:
            serv_sock.close()

    def serve_client(self, conn):
        try:
            req = self.parse_request(conn)
            resp = self.handle_request(req)
            self.send_response(conn, resp)
        except ConnectionResetError:
            conn = None
        except Exception as e:
            self.send_error(conn, e)

        if conn:
            conn.close()

    def parse_request(self, conn):
        """Разбор запроса и возвращение словаря с данными"""
        rfile = conn.makefile('r', encoding='utf-8')
        request_line = rfile.readline().strip().split()
        if len(request_line) != 3:
            raise ValueError("Некорректный HTTP-запрос")

        method, url, version = request_line
        headers = {}
        while True:
            line = rfile.readline().strip()
            if not line:
                break
            if ':' in line:
                key, value = line.split(":", 1)
                headers[key.strip()] = value.strip()

        body = ''
        if method.upper() == 'POST':
            length = int(headers.get('Content-Length', 0))
            body = rfile.read(length)

        return {
            'method': method.upper(),
            'url': url,
            'version': version,
            'headers': headers,
            'body': body
        }

    def handle_request(self, req):
        """Обработка GET и POST"""
        method = req['method']
        url = req['url']
        body = req['body']

        # Разбор URL и GET-параметров
        if '?' in url:
            path, query_string = url.split('?', 1)
            params = dict(pair.split('=') for pair in query_string.split('&') if '=' in pair)
        else:
            path = url
            params = {}

        if method == 'GET':
            # Возвращаем HTML со всеми оценками и формой
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
            # Разбор тела POST
            post_params = dict(pair.split('=') for pair in body.split('&') if '=' in pair)
            discipline = post_params.get('discipline', 'Не указано')
            mark = post_params.get('mark', 'Не указано')
            self._grades.append({'discipline': discipline, 'mark': mark})
            return {
                'status': 200,
                'reason': 'OK',
                'body': "<h1>Данные успешно сохранены</h1><a href='/'>Вернуться</a>"
            }
        else:
            return {
                'status': 405,
                'reason': 'Method Not Allowed',
                'body': '<h1>Метод не поддерживается</h1>'
            }

    def send_response(self, conn, resp):
        """Отправка ответа клиенту"""
        status_line = f"HTTP/1.1 {resp['status']} {resp['reason']}\r\n"
        headers = f"Content-Type: text/html; charset=utf-8\r\nContent-Length: {len(resp['body'].encode('utf-8'))}\r\n\r\n"
        conn.sendall(status_line.encode('utf-8'))
        conn.sendall(headers.encode('utf-8'))
        conn.sendall(resp['body'].encode('utf-8'))

    def send_error(self, conn, err):
        """Отправка ошибки 500"""
        body = f"<h1>500 Internal Server Error</h1><p>{err}</p>"
        try:
            conn.sendall(f"HTTP/1.1 500 Internal Server Error\r\nContent-Length: {len(body.encode('utf-8'))}\r\n\r\n".encode('utf-8'))
            conn.sendall(body.encode('utf-8'))
        except:
            pass

if __name__ == '__main__':
    host = sys.argv[1]
    port = int(sys.argv[2])
    name = sys.argv[3]

    serv = MyHTTPServer(host, port, name)
    try:
        serv.serve_forever()
    except KeyboardInterrupt:
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