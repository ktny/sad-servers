# "Salta": Docker container won't start.

## 問題
`/home/admin/app`配下にWebアプリケーションプログラムやDokckerfileがあるので、Dockerコンテナを立ち上げて8888ポートにcurlしてHello worldというレスポンスを受け取れるようにしたい。

## 解答

まずcurlすると8888ポートではすでにnginxが起動しているのでnginxを止める。

```sh
$ curl -v //localhost:8888

admin@ip-172-31-35-11:/$ curl -v localhost:8888
*   Trying 127.0.0.1:8888...
* Connected to localhost (127.0.0.1) port 8888 (#0)
> GET / HTTP/1.1
> Host: localhost:8888
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.18.0
< Date: Sun, 15 Jan 2023 00:37:50 GMT
< Content-Type: text/html
< Content-Length: 44
< Last-Modified: Fri, 16 Sep 2022 21:03:18 GMT
< Connection: keep-alive
< ETag: "6324e496-2c"
< Accept-Ranges: bytes
< 
these are not the droids you're looking for
* Connection #0 to host localhost left intact

# nginxをstop
$ sudo systemctl stop nginx
$ sudo systemctl status nginx
```

Dockerコンテナ/home/admin/app配下に移動しコンテナを立ち上げる準備をする。  
Dockerfileを確認すると8888ポートを開けていないのと、server.js
serve.jsを起動しているので修正する。

```Dockerfile
# port used by this app
EXPOSE 8888

# command to run
CMD [ "node", "server.js" ]
```

Dockerイメージのビルドとコンテナの起動を行う。
```sh
$ sudo docker build . -t salta
$ sudo docker run -d --rm -p 8888:8888 salta
$ curl localhost:8888
Hello World!
```
