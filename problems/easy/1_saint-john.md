# "Saint John": what is writing to this log file?

## 問題
`/var/log/bad.log`にログが書き込みされ続けてディスクを圧迫するのでそれがされないようにしたい。

## 解法
`lsof /var/log/bad.log`でファイルを対象としているプロセスがわかるのでそのプロセスをkillして終了。

```sh
$ lsof /var/log/bad.log
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF  NODE NAME
badlog.py 618 ubuntu    3w   REG  259,1    10671 67701 /var/log/bad.log

$ kill -9 618
```
