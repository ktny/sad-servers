# "Melbourne": WSGI with Gunicorn

## 問題

nginx->gunicorn->wsgi.pyのようなwebアプリケーションのインターフェースがある。  
nginxとgunicornはsystemdによって動いている。wsgi.pyは`/home/admin/wsgi.py`にある。  
これらの仕組みを使って`curl -s http://localhost`をすることで`Hello, world!`を返すようにしたい。

## 解法

curlをして80番ポートを確認すると受け付けてないことがわかる。

```sh
$ curl -v http://localhost
*   Trying 127.0.0.1:80...
* connect to 127.0.0.1 port 80 failed: Connection refused
* Failed to connect to localhost port 80: Connection refused
* Closing connection 0
curl: (7) Failed to connect to localhost port 80: Connection refused
```

nginxのステータスを確認すると立ち上がってないので立ち上げる。

```sh
$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:nginx(8)
$ sudo systemctl start nginx
```

再度curlすると80番ポートにはアクセスできたものの502エラー。

```sh
$ curl -v http://localhost
*   Trying 127.0.0.1:80...
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 502 Bad Gateway
< Server: nginx/1.18.0
< Date: Sun, 15 Jan 2023 08:23:19 GMT
< Content-Type: text/html
< Content-Length: 157
< Connection: keep-alive
< 
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.18.0</center>
</body>
</html>
* Connection #0 to host localhost left intact
```

nginxで`/`にリクエストしたときのルーティング先を調べると`/run/gunicorn.socket`にproxyしていることがわかる。

```sh
$ cat /etc/nginx/sites-enabled/default 
server {
    listen 80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.socket;
    }
}
```

gunicornのsystemdファイルなどを確認すると`gunicorn.socket`ではなく`gunicorn.sock`で動いていることがわかる。

```sh
# gunicornのsystemdファイルを確認する
$ cat /etc/systemd/system/gunicorn.service 
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=admin
Group=admin
WorkingDirectory=/home/admin
ExecStart=/usr/local/bin/gunicorn \
          --bind unix:/run/gunicorn.sock \
          wsgi
Restart=on-failure

[Install]
WantedBy=multi-user.target

# プロセスでgunicornの動いているsocketファイルを調べる
$ ps auxf | grep gunicorn
admin        920  0.0  0.1   5268   636 pts/0    S<+  08:27   0:00      \_ grep gunicorn
admin        612  0.0  4.7  27480 22124 ?        Ss   08:21   0:00 /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi
admin        678  0.0  4.1  27560 19376 ?        S    08:21   0:00  \_ /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi
```

nginx設定を`gunicorn.sock`に修正してrestartする。

```sh
$ sudo vi /etc/nginx/sites-enabled/default 
$ sudo systemctl restart nginx
```

再度curlすると200だが`Content-Length: 0`でなにも返ってきていない。

```sh
$ curl -v http://localhost
*   Trying 127.0.0.1:80...
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.18.0
< Date: Sun, 15 Jan 2023 08:29:44 GMT
< Content-Type: text/html
< Content-Length: 0
< Connection: keep-alive
< 
* Connection #0 to host localhost left intact
```

UNIXドメインソケットを明示的に通してアクセスするとgunicornが返してくるのも`Content-Length: 0`なので、nginxではなくgunicornより先に原因があることがわかる。

```sh
$ curl -I --unix-socket /run/gunicorn.sock http://localhost
HTTP/1.1 200 OK
Server: gunicorn
Date: Sun, 15 Jan 2023 08:30:46 GMT
Connection: close
Content-Type: text/html
Content-Length: 0
```

`/home/admin/wsgi.py`を確認するとレスポンスヘッダで`Content-Length: 0`を返すようにしている。

```sh
$ cat /home/admin/wsgi.py 
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html'), ('Content-Length', '0'), ])
    return [b'Hello, world!']
```

コンテンツを返せるように`Content-Length: 20`などにしてgunicornを再起動する。

```sh
$ vi /home/admin/wsgi.py 
$ sudo systemctl restart gunicorn
$ curl -s localhost
Hello, world!
```
