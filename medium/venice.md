# "Venice": Am I in a container?

## 問題

現在のサーバがコンテナ上かどうか調べたい。

## 解法

### 参考
https://stackoverflow.com/questions/23513045/how-to-check-if-a-process-is-running-inside-docker-container

コンテナ上ではpid=1のプロセスは通常のホストのpidとは違いがあるのでそこを見る。

```sh
# 問題のサーバ上では/で終わっているcgroupがないのでコンテナであると言える。通常のホストではすべて/で終わる
# Dockerコンテナであれば、/docker/配下のcgroupが存在する
$ cat /proc/1/cgroup
0::/init.scope
```

また、ルートディレクトリのinode番号でも確認することができる。  
通常のホストであればルートディレ栗のinode番号は2なので、それ以外であればコンテナであると言える。

```sh
$ cd /
$ ls -ali
total 72
277640 dr-xr-xr-x   1 root root 4096 Jan 15 04:28 . # inode番号が277640なのでコンテナ内
277640 dr-xr-xr-x   1 root root 4096 Jan 15 04:28 ..
287384 drwxr-xr-x   1 root root 4096 Sep 24 22:36 bin
277411 drwxr-xr-x   2 root root 4096 Sep  3 12:10 boot
     1 drwxr-xr-x  10 root root 2660 Jan 15 04:28 dev
290181 drwxr-xr-x   1 root root 4096 Jan 15 04:28 etc
396395 drwxr-xr-x   2 root root 4096 Sep  3 12:10 home
420788 drwxr-xr-x   1 root root 4096 Sep 24 22:36 lib
396603 drwxr-xr-x   2 root root 4096 Sep 12 00:00 lib64
396605 drwxr-xr-x   2 root root 4096 Sep 12 00:00 media
396606 drwxr-xr-x   2 root root 4096 Sep 12 00:00 mnt
396607 drwxr-xr-x   2 root root 4096 Sep 12 00:00 opt
     1 dr-xr-xr-x 128 root root    0 Jan 15 04:28 proc
396609 drwx------   2 root root 4096 Sep 12 00:00 root
     1 drwxrwxrwt  11 root root  320 Jan 15 04:28 run
 30311 drwxr-xr-x   1 root root 4096 Sep 24 22:36 sbin
396677 drwxr-xr-x   2 root root 4096 Sep 12 00:00 srv
     1 dr-xr-xr-x  13 root root    0 Jan 15 04:27 sys
     1 drwxrwxrwt   8 root root  160 Jan 15 04:28 tmp
290133 drwxr-xr-x   1 root root 4096 Sep 12 00:00 usr
 31974 drwxr-xr-x   1 root root 4096 Sep 12 00:00 var
```
