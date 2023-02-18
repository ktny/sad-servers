# "Jakarta": it's always DNS.

## 問題

`ping google.com`としても`ping: google.com: Name or service not known`と出てホストの名前解決ができない。  
解決できるようにしたい。

## 解法

`/etc/resolve.conf`でDNSサーバが設定されているかを確認する。  
とりあえずは設定されていそう。

```conf
nameserver 127.0.0.53
options edns0 trust-ad
search us-east-2.compute.internal
```

`/etc/nsswitch.conf`でホスト名解決の優先順位を確認する。  
`hosts: files`となっているので`/etc/hosts`での解決はできているがDNSでの解決ができていない。  
`hosts: files dns`に修正してDNSで解決できるようにする。

```conf
passwd:         files systemd
group:          files systemd
shadow:         files
gshadow:        files

hosts:          files
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

pingを打つと名前解決ができる（外部接続できないので名前解決はできてもレスポンスは返ってこない）。

```sh
$ ping google.com
PING google.com (142.250.190.78) 56(84) bytes of data.
```
