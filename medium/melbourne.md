# "Melbourne": WSGI with Gunicorn

## 問題
nginx->gunicorn->wsgi.pyのようなwebアプリケーションのインターフェースがある。  
nginxとgunicornはsystemdによって動いている。wsgi.pyは`/home/admin/wsgi.py`にある。  
これらの仕組みを使って`curl -s http://localhost`をすることで`Hello, world!`を返すようにしたい。

## 解法
