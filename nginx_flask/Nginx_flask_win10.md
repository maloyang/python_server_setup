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

## 改用一個實際的flask專案進行測試
先說測試結果，可能是我對Nginx還不熟，目前感覺是由flask在提供static資料的服務，而不是由Nginx提供服務

- tonado的server.py配置
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

- 因為flask的app.py比較大，所以只列出一部份，其他略過
```python
import os
import random
import requests
import datetime, time
import re
import logging
import ftplib
import csv
import hashlib

from functools import wraps
from flask import Flask, request, abort, render_template, Response
from flask import json, jsonify, session, redirect, url_for
from flask_cors import CORS, cross_origin # for cross domain problem
from flask import send_file

#... 中間略

app = Flask(__name__)
#app = Flask(__name__, static_url_path='', static_folder='static')

#... 中間略

if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=5000)

```

- 讓tonado作為 WSGI 把 flask app帶起來
python server.py

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
            # Malo add: (flask_2)
            #root D:\nginx\flask_2;
            #proxy_pass http://localhost:5000; 
            # Malo add: (wb_watergate)
            root D:\nginx\wb_watergate\static;
            proxy_pass http://localhost:5000;
        }

```

- 啟動 nginx.exe
start nginx.exe

- 這樣就可以在 http://127.0.0.1:8080/ 的連結轉到 5000 port的flask服務的內容

### 說明一下這邊我有做兩種測試
- 1.首先是如上的flask的app.py的寫法

- 這時，我要取得 ./static/index.html 的時候，會是使用 http://127.0.0.1:8080/static/index.html
- 但是我本來的預期是由Nginx接手處理靜態檔，```root D:\nginx\wb_watergate\static;```這一段設定應該要生效才對

- 2.然後，我再把app.py中的這一段改成這樣
```python
#app = Flask(__name__)
app = Flask(__name__, static_url_path='', static_folder='static')
```

- 這時，我只要用 http://127.0.0.1:8080/index.html 就可以取得 ./static/index.html 的東西了，所以確實是由flask服務才對

### 等我再對Nginx熟一點，多做點測試再來更新 (2022-04-23)

