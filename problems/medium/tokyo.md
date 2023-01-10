# "Tokyo": can't serve web file

## 問題
`hello sadserver`と書かれた`/var/www/html/index.html`をwebサーバで配信しているが`curl 127.0.0.1:80`してもレスポンスが来ないので返ってくるようにしたい。

## 解法

試しに`curl`してみる。
```sh
# レスポンスが来ないことが確認できる
$ curl 127.0.0.1:80

# 80番をLISTENしているプロセスを確認するとapache2が動いていることがわかる 
$ lsof -i:80
COMMAND PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
apache2 643     root    4u  IPv6  16815      0t0  TCP *:http (LISTEN)
apache2 775 www-data    4u  IPv6  16815      0t0  TCP *:http (LISTEN)
apache2 776 www-data    4u  IPv6  16815      0t0  TCP *:http (LISTEN)
```

80番がブロックされているか確認。
INPUTで全ポートに対するDROPルールがあるため`iptables -F`でクリアする。
```sh
$ iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  anywhere             anywhere             tcp dpt:http

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

$ iptables -F
```

もう一度`curl`すると権限がないと言われる。
```sh
$ curl 127.0.0.1:80
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.52 (Ubuntu) Server at 127.0.0.1 Port 80</address>
</body></html>
```

ファイルの権限を見るとowner(root)にしか読み取り権限がないので`chmod 644`でその他ユーザーにも読み取り権限を与える。
```sh
root@ip-172-31-21-14:/# ls -la /var/www/html/index.html 
-rw------- 1 root root 16 Aug  1 00:40 /var/www/html/index.html
root@ip-172-31-21-14:/# chmod 644 /var/www/html/index.html
root@ip-172-31-21-14:/# curl 127.0.0.1:80
hello sadserver
```
