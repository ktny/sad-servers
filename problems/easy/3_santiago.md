# "Santiago": Find the secret combination

## 問題
次の2つの数字を見つけ出したい。
1. `/home/admin`ディレクトリの`*.txt`にある`Alice`が出現する行数
2. `Alice`が1回しか出てこないファイルがある。その`Alice`が出現する次の行に書かれている数値

## 解法

```sh
# 指定のファイル内でAliceが出現する行数
$ grep Alice /home/admin/*.txt | wc -l
411

# Aliceが1回しか出てこないファイルを特定
$ grep -c Alice *.txt
11-0.txt:398
1342-0.txt:1
1661-0.txt:12
84-0.txt:0

# そのファイルでAliceが出てくる行と前後1行を表示
$ grep -1 Alice 1342-0.txt
                              PUBLISHER
                                Alice
                        156 CHARING CROSS ROAD

$ echo -n 411 > /home/admin/solution; echo 156 >> /home/admin/solution
```
