# Nginx + flask + Win10 (中間還有層tonado)

紀錄在 Win10 下佈署 Nginex + flask 的過程
ref: 參考: https://segmentfault.com/a/1190000040405656

## 基本配置範例

- 建立run.py
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

- 建立 server.py
```python
from tornado.wsgi import WSGIContainer
from tornado.httpserver import HTTPServer
from tornado.ioloop import IOLoop
from app import app

http_server = HTTPServer(WSGIContainer(app))
http_server.listen(5000)
print('run...')
IOLoop.current().start()
```

- 讓tonado作為 WSGI 把 flask app帶起來
python server.py

---

### 設定 Nginx
- 設定 nginx.exe的nginx.conf
```conf
    server {
        listen       8080;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            # 原來的
            #root   html;
            #index  index.html index.htm;
            # Malo add:
            root D:\nginx\flask_2;
            proxy_pass http://localhost:5000; 
        }
```

- 啟動 nginx.exe
start nginx.exe

- 這樣就可以在 http://127.0.0.1:8080/ 的連結轉到 5000 port的flask服務的內容

----
