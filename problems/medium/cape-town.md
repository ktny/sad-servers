# "Cape Town": Borked Nginx

## 問題
Nginxに`curl -I 127.0.0.1:80`としても次のようなエラーが出るのを直したい。
`curl: (7) Failed to connect to localhost port 80: Connection refused`

## 解法

nginxのステータスを確認。
`/etc/nginx/sites-enabled/default`の1行目に`;`があってエラーになっていることがわかるので削除。
```sh
$ systemctl status nginx
● nginx.service - The NGINX HTTP and reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Mon 2023-01-09 13:01:25 UTC; 30s ago
    Process: 581 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
        CPU: 33ms

Jan 09 13:01:24 ip-172-31-34-5 systemd[1]: Starting The NGINX HTTP and reverse proxy server...
Jan 09 13:01:25 ip-172-31-34-5 nginx[581]: nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/default:1
Jan 09 13:01:25 ip-172-31-34-5 nginx[581]: nginx: configuration file /etc/nginx/nginx.conf test failed
Jan 09 13:01:25 ip-172-31-34-5 systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Jan 09 13:01:25 ip-172-31-34-5 systemd[1]: nginx.service: Failed with result 'exit-code'.
Jan 09 13:01:25 ip-172-31-34-5 systemd[1]: Failed to start The NGINX HTTP and reverse proxy server.
```

再起動してcurlするも500エラー。
```sh
$ sudo systemctl restart nginx
$ curl -I 127.0.0.1:80
HTTP/1.1 500 Internal Server Error
```

エラーログを確認するとToo many open filesのエラーが出ている。
nginxは1プロセスで多くのアクセスを捌くのでファイルを開く数（ファイルディスクリプタ）が多くなるが多くなりすぎるとエラーになる。
```sh
$ less /var/log/nginx/error.log 
2022/09/11 16:26:17 [crit] 5801#5801: accept4() failed (24: Too many open files)
2022/09/11 16:26:17 [crit] 5801#5801: *2 open() "/var/www/html/index.nginx-debian.html" failed (24: Too many open files), client: 127.0.0.1, server: _, request: "GET / HTTP/1.1", host: "localhost:80"
```

ファイルディスクリプタはnginxの設定（`/etc/nginx/nginx.conf`など）の`worker_rlimit_nofile`でも設定されることがあるが、今回はsystemd側で過度に低い値が設定されていたようなのでコメントアウトする。
```sh
$ cat /etc/systemd/system/nginx.service
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
#LimitNOFILE=10

[Install]
```

nginxを再起動してcurlを確認。
```sh
$ sudo systemctl restart nginx 
Warning: The unit file, source configuration file or drop-ins of nginx.service changed on disk. Run 'systemctl daemon-reload' to reload units.

$ sudo systemctl daemon-reload
$ sudo systemctl restart nginx
$ curl -I 127.0.0.1:80
HTTP/1.1 200 OK
```
