# "Manhattan": can't write data into database.

## 問題
PostgreSQLに行をインサートしようとしてもできなくなっているのでできるようにしたい。

## 解法
実際にINSERTを行いPostgreSQLのログを確認する。
`No space left on device`とあるのでディスク空き容量が足りないことがわかる。

```sh
$ sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
        Is the server running locally and accepting connections on that socket?

# /var/log/syslogを見ても同様のエラーが出ていることがわかる
$ less /var/log/postgresql/postgresql-14-main.log.1
2023-01-08 05:15:39.112 UTC [646] FATAL:  could not create lock file "postmaster.pid": No space left on device
pg_ctl: could not start server
Examine the log output.
```

`df`コマンドでディスク使用量を確認すると`/opt/pgdata`が100%使用されている。
```sh
$ df -h
Filesystem       Size  Used Avail Use% Mounted on
udev             224M     0  224M   0% /dev
tmpfs             47M  1.5M   46M   4% /run
/dev/nvme1n1p1   7.7G  1.2G  6.1G  17% /
tmpfs            233M     0  233M   0% /dev/shm
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            233M     0  233M   0% /sys/fs/cgroup
/dev/nvme1n1p15  124M  278K  124M   1% /boot/efi
/dev/nvme0n1     8.0G  8.0G   28K 100% /opt/pgdata
```

実際に`/opt/pgdata`を確認するとバックアップファイルが大半を占めていることが確認できるので、削除してPostgreSQLを再起動する。
```sh
$ ls -lah /opt/pgdata
total 8.0G
drwxr-xr-x  3 postgres postgres   82 May 21  2022 .
drwxr-xr-x  3 root     root     4.0K May 21  2022 ..
-rw-r--r--  1 root     root       69 May 21  2022 deleteme
-rw-r--r--  1 root     root     7.0G May 21  2022 file1.bk
-rw-r--r--  1 root     root     923M May 21  2022 file2.bk
-rw-r--r--  1 root     root     488K May 21  2022 file3.bk
drwx------ 19 postgres postgres 4.0K May 21  2022 main

$ rm /opt/pgdata/*.bk
$ systemctl restart postgresql
$ sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
INSERT 0 1
```
