# "Lisbon": etcd SSL cert troubles

## 問題

etcdサーバがlocalhost:2379で稼働しているのでkey=fooでbarを返すようにしたい。  
etcdサーバへのリクエストは`etcdctl get foo`か`curl https://localhost:2379/v2/keys/foo`でできる。

## 解法

etcdeとは設定情報の共有のための分散KVS。  
試しに`etcdctl get foo`とするとcurrent timeが1年後であるというエラーが返ってきた。

```sh
$ etcdctl get foo
Error:  client: etcd cluster is unavailable or misconfigured; error #0: x509: certificate has expired or is not yet valid: current time 2024-01-28T00:50:13Z is after 2023-01-30T00:02:48Z

error #0: x509: certificate has expired or is not yet valid: current time 2024-01-28T00:50:13Z is after 2023-01-30T00:02:48Z
```

`data -s`コマンドでサーバの現在日時を修正すると別のエラーが返ってくるようになった。

```sh
$ date
Sun Jan 28 00:51:21 UTC 2024
$ sudo date -s "last year"
Sat Jan 28 00:51:37 UTC 2023
$ etcdctl get foo
Error:  client: response is invalid json. The endpoint is probably not valid etcd cluster endpoint.
```

etcdのエンドポイントに届いていない可能性がある。
プロセスを確認するとetcd自体は2379ポートをlistenしていそうなので、2379ポートへのリクエストが別のところにリダイレクトされていないかを確認する。

```sh
$ ps aux  | grep etcd
etcd         587  0.6  5.0 11729040 23664 ?      Ssl  01:32   0:01 /usr/bin/etcd --cert-file /etc/ssl/certs/localhost.crt --key-file /etc/ssl/certs/localhost.key --advertise-client-urls=https://localhost:2379 --listen-client-urls=https://localhost:2379
```

iptablesを見ると2379ポートが443ポートへリダイレクトされている設定があるのでこれを削除する。

```sh
# natテーブルを確認
$ sudo iptables -t nat -L --line-numbers
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination         
1    DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:2379 redir ports 443
2    DOCKER     all  --  anywhere            !ip-127-0-0-0.us-east-2.compute.internal/8  ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
num  target     prot opt source               destination         
1    MASQUERADE  all  --  ip-172-17-0-0.us-east-2.compute.internal/16  anywhere            

Chain DOCKER (2 references)
num  target     prot opt source               destination         
1    RETURN     all  --  anywhere             anywhere 

# ルールを削除
$ sudo iptables -t nat -D OUTPUT 1

$ etcdctl get foo
bar
```
