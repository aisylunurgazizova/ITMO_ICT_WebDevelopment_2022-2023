Web-программирование 2022
========================
Нургазизова Айсылу, K33422
-------------------------
Лабораторная работа 1.
1 задание.
Реализовать клиентскую и серверную часть приложения. Клиент отсылает серверу
сообщение «Hello, server». Сообщение должно отразиться на стороне сервера.
Сервер в ответ отсылает клиенту сообщение «Hello, client». Сообщение должно
отобразиться у клиента.
- client.py 
```python
import socket
conn = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
conn.connect(("127.0.0.1", 8080))
conn.send(b"Hello, server! \n")
data = conn.recv(1024)
print(data.decode("utf-8") )
conn.close()
```
- server.py
```python
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.bind(("127.0.0.1", 8080))
sock.listen(10)
while True:
    conn, addr = sock.accept()
    data = conn.recv(1024)
    udata = data.decode("utf-8")
    print(udata)
    if not data:
        break
    conn.send(b"Hello, client! \n")
conn.close()
```
2 задание.
Реализовать клиентскую и серверную часть приложения. Клиент запрашивает у
сервера выполнение математической операции, параметры, которые вводятся с
клавиатуры. Сервер обрабатывает полученные данные и возвращает результат
клиенту. Мой вариант: площадь параллелограмма.
- client.py
```python
import socket
conn = socket.socket()
conn.connect(("127.0.0.1", 8080))
a = input("Введите высоту: ")
b = input("Введите основание: ")
conn.send(a.encode())
conn.send(b.encode())
c_bin = conn.recv(2000)
c = c_bin.decode('utf-8')
print('Площадь параллелограмма: ', c)
conn.close()
```
- server.py
```python
import socket
server = socket.socket()
host = '127.0.0.1'
port = 8080
server.bind((host, port))
server.listen(3)
conn, addr = server.accept()
a_bin = conn.recv(2000)
b_bin = conn.recv(2000)
a = a_bin.decode('utf-8')
b = b_bin.decode('utf-8')
c = int(a) * int(b)
conn.send(str(c).encode())
conn.close()
```
3 задание.
Реализовать серверную часть приложения. Клиент подключается к серверу. В ответ
клиент получает http-сообщение, содержащее html-страницу, которую сервер
подгружает из файла index.html.
- client.py
```python
import socket
conn = socket.socket()
conn.connect(("127.0.0.1", 8080))
result = conn.recv(10000)
print(result.decode())
conn.close()
```
- server.py
```python
import socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
host = '127.0.0.1'
port = 8080
server.bind((host, port))
server.listen(1)
while True:
    conn, addr = server.accept()
    page = open('index.html')
    content = page.read()
    page.close()
    response = 'HTTP/1.0 200 OK\n\n' + content
    conn.sendall(response.encode())
    conn.close()
```
- index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Коты</title>
</head>
<body>
    <p>
        Кошка (лат. Felis catus) — домашнее животное, одно из наиболее популярных (наряду с собакой) «животных-компаньонов».
    </p>
    <p>
        <img src="https://murkoshka.ru/wp-content/uploads/NeedFull.NET_porody_koshek_i_kotov_cr.jpg" width="30%" align="center" border="3" hspace="10%" vspace="10%" />
    </p>
</body>
</html>
```

4 задание.
Реализовать двухпользовательский или многопользовательский чат.
- client.py
```python
import socket
from threading import Thread
def send_message():
    while True:
        text = input()
        sock.sendall(bytes(text, "utf-8"))
        if text == "q":
            sock.close()
            break
def receive_message():
    try:
        while True:
            data = sock.recv(1024)
            udata = data.decode("utf-8")
            print(udata)
    except ConnectionAbortedError:
        print('You left the chat')
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('localhost', 55551))
send_thread = Thread(target=send_message)
get_thread = Thread(target=receive_message)
send_thread.start()
get_thread.start()
```
- client1.py
```python
import socket
from threading import Thread
def send_message():
    while True:
        text = input()
        sock.sendall(bytes(text, "utf-8"))
        if text == "q":
            sock.close()
            break
def receive_message():
    try:
        while True:
            data = sock.recv(1024)
            udata = data.decode("utf-8")
            if udata != 'q':
                print(udata)
    except ConnectionAbortedError as e:
        print('You left the chat')
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('localhost', 55551))
send_thread = Thread(target=send_message)
get_thread = Thread(target=receive_message)
send_thread.start()
get_thread.start()
```
- server.py
```python
import socket
from threading import Thread
users = []
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.bind(('', 55551))
sock.listen(10)
sock.setblocking(False)
def chat_users():
    while True:
        sock.setblocking(True)
        con, addr = sock.accept()
        sock.setblocking(False)
        con.send(b'Enter your name:  \n')
        name = con.recv(1024)
        con.send(b'You can send messages now. Press q if you want to leave the chat.  \n')
        name = name.decode("utf-8")
        print(name, 'has joined the chat')
        if con not in users:
            users.append((con, name))
def message():
    while True:
        try:
            for user in users:
                text = user[0].recv(1024).decode('utf-8')
                if text == "q":
                    user[0].close()
                    print('{} has left the chat'.format(user[1]))
                else:
                    print(str(user[1]) + ':' + text)
                for send_user in users:
                    if send_user[0] != user[0]:
                        data = f'{user[1]}: ' + text
                        send_user[0].sendall(data.encode('utf8'))
        except socket.error:
            print('No one is in the chat. Waiting for new users...')
            break
user_thread = Thread(target=chat_users)
message_thread = Thread(target=message)
user_thread.start()
message_thread.start()
```

5 задание.
Необходимо написать простой web-сервер для обработки GET и POST http
запросов средствами Python и библиотеки socket.
- server.py
```python
import socket
import sys
MAX_LINE = 64 * 1024
MAX_HEADERS = 100
class MyHTTPServer:
    def __init__(self, host, port):
        self._host = host
        self._port = port
        self._subjs = []
        self._marks = []
    def serve_forever(self):
        serv_sock = socket.socket(
            socket.AF_INET,
            socket.SOCK_STREAM,
            proto=0)
        try:
            serv_sock.bind((self._host, self._port))
            serv_sock.listen()
            while True:
                conn, _ = serv_sock.accept()
                try:
                    self.serve_client(conn)
                except Exception as e:
                    print('Client serving failed', e)
        finally:
            serv_sock.close()
    def serve_client(self, conn):
        data = conn.recv(16384)
        data = data.decode('utf-8')
        url, method, headers, body = self.parse_request(data)
        resp = self.handle_request(url, method, body)
        if resp:
            self.send_response(conn, resp)
    def parse_request(self, data):
        data = data.replace('\r', '')
        lines = data.split('\n')
        method, url, protocol = lines[0].split()
        end_headers = lines.index('')
        headers = lines[1:end_headers]
        body = lines[-1]
        return url, method, headers, body
    def handle_request(self, url, method, body):
        if url == '/':
            if method == 'GET':
                resp = "HTTP/1.1 200 OK\n\n"
                with open('index.html', 'r') as file:
                    resp += file.read()
                return resp
            if method == 'POST':
                resp = "HTTP/1.1 204 OK\n\n"
                params = body.split('&')
                for a in params:
                    if a.split('=')[0] == 'subj':
                        self._subjs.append(a.split('=')[1])
                    if a.split('=')[0] == 'mark':
                        self._marks.append(a.split('=')[1])

                resp += "<html><head><title>Journal</title></head><body><ol>"
                for s, m in zip(self._subjs, self._marks):
                    resp += f"<li>Subject: {s}, Grade: {m}</li>"
                resp += "</ol></body></html>"
                return resp
    def send_response(self, conn, resp):
        conn.send(resp.encode('utf-8'))
if __name__ == '__main__':
    host ='127.0.0.1'
    port = 8000
    serv = MyHTTPServer(host, port)
    try:
        serv.serve_forever()
    except KeyboardInterrupt:
        pass
```

- index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Journal</title>
</head>
<body>
    <p>Enter data regarding grades
        <form action="/" method="post">
            <label for="subject">Subject:</label>
            <input type="text" name="subject" id="subject"/>
            <label for="grade">Grade:</label>
            <input type="number" name="grade" id="grade"/>
            <button>Send</button>
        </form>
    </p>
</body>
</html>
```