# "Oaxaca": Close an Open File

## 問題
`/home/admin/somefile`があるプロセスから開かれている。プロセスをkillすることなくファイルを閉じたい。

## 解法

`lsof`をするとファイルディスクリプタ番号が77であり書き込み可能状態であることがわかる。  
ファイルディスクリプタとはプログラムが開いているファイルの識別番号。

```sh
$ lsof /home/admin/somefile
COMMAND PID  USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
bash    803 admin   77w   REG  259,1        0 272875 /home/admin/somefile
```

プロセスをkillしてはいけないので、ファイルディスクリプタからcloseする。  
execのマニュアルによると`exec [file descriptor number]<&-`でcloseできることがわかる。

https://man7.org/linux/man-pages/man1/exec.1p.html

> Close file descriptor 3:  
> exec 3<&-

```sh
$ exec 77<&-
$ lsof /home/admin/somefile
```
