# "Saskatoon": counting IPs.

## 問題
`/home/admin/access.log`にwebサーバのアクセスログがある。このファイル中で最もアクセスが多かったIP元を`/home/admin/highestip.txt`に記録したい。

## 解法
`cut`でIP部分だけを抜き出し、`sort` `uniq`で重複しているIPを集計する。
`uniq`は連続している重複を取り除くので前段に`sort`が必要なことに注意。
`cut -d " " -f 1`でスペース区切りの先頭カラムのみ取り出している。これは`awk '{print  $1}'`でもできる。

```sh
$ cat /home/admin/access.log | cut -d " " -f 1 | sort | uniq -c | sort | tail
     82 198.46.149.143
     83 208.115.111.72
     84 100.43.83.137
     99 68.180.224.225
    102 209.85.238.199
    113 50.16.19.13
    273 75.97.9.59
    357 130.237.218.86
    364 46.105.14.53
    482 66.249.73.135

$ echo "66.249.73.135" > /home/admin/highestip.txt
```
