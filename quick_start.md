
<!-- TOC -->

- [awk概览](#awk概览)
    - [1.起步上台`](#1起步上台)
    - [2. 过滤记录](#2-过滤记录)
    - [3. 内建变量](#3-内建变量)
        - [3.1 指定分隔符](#31-指定分隔符)
    - [3. 字符串匹配](#3-字符串匹配)
        - [3.1 折分文件](#31-折分文件)
    - [4. 统计](#4-统计)
    - [5. awk 脚本](#5-awk-脚本)
    - [环境变量](#环境变量)
    - [6. 几个花活](#6-几个花活)
        - [6.1 从file文件中找出长度大于80的行](#61-从file文件中找出长度大于80的行)
        - [6.2 按连接数查看客户端IP](#62-按连接数查看客户端ip)
        - [6.3 打印九九乘法表](#63-打印九九乘法表)

<!-- /TOC -->

# awk概览

awk是Unix最古老的命令,当前的测试数据为data/netstat.txt

## 1.起步上台`

下面是最简单最常用的awk示例，其输出第1列和第4例，

* 其中单引号中的被大括号括着的就是awk的语句，注意，其只能被单引号包含。
* 其中的$1..$n表示第几例。注：$0表示整个行。

```bash
$ awk '{print $1, $4}' netstat.txt

Proto Local-Address
tcp 0.0.0.0:3306
tcp 0.0.0.0:80
tcp 127.0.0.1:9000
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp :::22
```

* 我们再来看看awk的格式化输出，和C语言的printf没什么两样：

```bash
awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}' netstat.txt

Proto    Recv-Q   Send-Q   Local-Address      Foreign-Address        State
tcp      0        0        0.0.0.0:3306       0.0.0.0:*              LISTEN
tcp      0        0        0.0.0.0:80         0.0.0.0:*              LISTEN
tcp      0        0        127.0.0.1:9000     0.0.0.0:*              LISTEN
tcp      0        0        coolshell.cn:80    124.205.5.146:18245    TIME_WAIT
tcp      0        0        coolshell.cn:80    61.140.101.185:37538   FIN_WAIT2
tcp      0        0        coolshell.cn:80    110.194.134.189:1032   ESTABLISHED
tcp      0        0        coolshell.cn:80    123.169.124.111:49809  ESTABLISHED
tcp      0        0        coolshell.cn:80    116.234.127.77:11502   FIN_WAIT2
tcp      0        0        coolshell.cn:80    123.169.124.111:49829  ESTABLISHED
tcp      0        0        coolshell.cn:80    183.60.215.36:36970    TIME_WAIT
tcp      0        4166     coolshell.cn:80    61.148.242.38:30901    ESTABLISHED
tcp      0        1        coolshell.cn:80    124.152.181.209:26825  FIN_WAIT1
tcp      0        0        coolshell.cn:80    110.194.134.189:4796   ESTABLISHED
tcp      0        0        coolshell.cn:80    183.60.212.163:51082   TIME_WAIT
tcp      0        1        coolshell.cn:80    208.115.113.92:50601   LAST_ACK
tcp      0        0        coolshell.cn:80    123.169.124.111:49840  ESTABLISHED
tcp      0        0        coolshell.cn:80    117.136.20.85:50025    FIN_WAIT2
tcp      0        0        :::22              :::*
```

## 2. 过滤记录

我们再来看看如何过滤记录（下面过滤条件为：第三列的值为0 && 第6列的值为LISTEN）

``` bash
$ awk '$3==0 && $6=="LISTEN" ' netstat.txt

tcp        0      0 0.0.0.0:3306           0.0.0.0:*                   LISTEN
tcp        0      0 0.0.0.0:80             0.0.0.0:*                   LISTEN
tcp        0      0 127.0.0.1:9000         0.0.0.0:*                   LISTEN
```

* 我们来看看各种过滤记录的方式：

```bash
$ awk ' $3>0 {print $0}' netstat.txt
Proto Recv-Q Send-Q Local-Address          Foreign-Address             State
tcp        0   4166 coolshell.cn:80        61.148.242.38:30901         ESTABLISHED
tcp        0      1 coolshell.cn:80        124.152.181.209:26825       FIN_WAIT1
tcp        0      1 coolshell.cn:80        208.115.113.92:50601        LAST_ACK
```

* 如果我们需要表头的话，我们可以引入内建变量NR：

```bash
$ awk '$3==0 && $6=="LISTEN" || NR==1 ' netstat.txt
Proto Recv-Q Send-Q Local-Address          Foreign-Address             State
tcp        0      0 0.0.0.0:3306           0.0.0.0:*                   LISTEN
tcp        0      0 0.0.0.0:80             0.0.0.0:*                   LISTEN
tcp        0      0 127.0.0.1:9000         0.0.0.0:*                   LISTEN
```

* 再加上格式化输出：

```bash
$ awk '$3==0 && $6=="LISTEN" || NR==1 {printf "%-20s %-20s %s\n",$4,$5,$6}' netstat.txt
Local-Address        Foreign-Address      State
0.0.0.0:3306         0.0.0.0:*            LISTEN
0.0.0.0:80           0.0.0.0:*            LISTEN
127.0.0.1:9000       0.0.0.0:*            LISTEN
```

## 3. 内建变量

说到了内建变量，我们可以来看看awk的一些内建变量：

|变量名|含义|
|---|---|
|$0|当前记录(当前变量中存在着整行的内容)|
|\$1~$n|当前记录的第n个字段，字段间由FS分隔|
|FS|输入字段分隔符 默认是空格或Tab|
|NF|当前记录中的字段个数，就是有多少列|
|NR|已经读出的记录数，就是行号，从1开始，如果有多个文件话，这个值也是不断累加中。|
|FNR|当前记录数，与NR不同的是，这个值会是各个文件自己的行号|
|RS|输入的记录分隔符， 默认为换行符|
|OFS|输出字段分隔符， 默认也是空格|
|ORS|输出的记录分隔符，默认为换行符|
|FILENAME|当前输入文件的名字|

* 怎么使用呢，比如：我们如果要输出行号：

```bash
$ awk '$3==0 && $6=="ESTABLISHED" || NR==1 {printf "%02s %s %-20s %-20s %s\n",NR, FNR, $4,$5,$6}' netstat.txt

01 1 Local-Address        Foreign-Address      State
07 7 coolshell.cn:80      110.194.134.189:1032 ESTABLISHED
08 8 coolshell.cn:80      123.169.124.111:49809 ESTABLISHED
10 10 coolshell.cn:80      123.169.124.111:49829 ESTABLISHED
14 14 coolshell.cn:80      110.194.134.189:4796 ESTABLISHED
17 17 coolshell.cn:80      123.169.124.111:49840 ESTABLISHED
```

### 3.1 指定分隔符

```bash
$ awk 'BEGIN{FS=":"} {print $1,$2,$3}' /etc/passwd
root 0 /root
bin 1 /bin
daemon 2 /sbin
adm 3 /var/adm
lp 4 /var/spool/lpd
sync 5 /sbin
shutdown 6 /sbin
halt 7 /sbin
```

* 上面的命令也等价于：（-F的意思就是指定分隔符）

```bash
awk -F: '{print $1,$2,$3}' /etc/passwd
```

* 注：如果你要指定多个分隔符，你可以这样来：

```bash
awk -F '[;:]'
```

* 再来看一个以\t作为分隔符输出的例子（下面使用了/etc/passwd文件，这个文件是以:分隔的）：

```bash
$ awk  -F: '{print $1,$3,$6}' OFS="\t" /etc/passwd
root    0       /root
bin     1       /bin
daemon  2       /sbin
adm     3       /var/adm
lp      4       /var/spool/lpd
sync    5       /sbin
```

## 3. 字符串匹配

我们再来看几个字符串匹配的示例

```bash
$ awk '$6 ~ /FIN/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt
1	Local-Address	Foreign-Address	State
6	coolshell.cn:80	61.140.101.185:37538	FIN_WAIT2
9	coolshell.cn:80	116.234.127.77:11502	FIN_WAIT2
13	coolshell.cn:80	124.152.181.209:26825	FIN_WAIT1
18	coolshell.cn:80	117.136.20.85:50025	FIN_WAIT2
```

```bash
$ awk '$6 ~ /WAIT/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt

1	Local-Address	Foreign-Address	State
5	coolshell.cn:80	124.205.5.146:18245	TIME_WAIT
6	coolshell.cn:80	61.140.101.185:37538	FIN_WAIT2
9	coolshell.cn:80	116.234.127.77:11502	FIN_WAIT2
11	coolshell.cn:80	183.60.215.36:36970	TIME_WAIT
13	coolshell.cn:80	124.152.181.209:26825	FIN_WAIT1
15	coolshell.cn:80	183.60.212.163:51082	TIME_WAIT
18	coolshell.cn:80	117.136.20.85:50025	FIN_WAIT2
```

上面的第一个示例匹配FIN状态， 第二个示例匹配WAIT字样的状态。其实 ~ 表示模式开始。/ /中是模式。这就是一个正则表达式的匹配。

其实awk可以像grep一样的去匹配第一行，就像这样：

```bash
$ awk '/LISTEN/' netstat.txt
tcp        0      0 0.0.0.0:3306           0.0.0.0:*                   LISTEN
tcp        0      0 0.0.0.0:80             0.0.0.0:*                   LISTEN
tcp        0      0 127.0.0.1:9000         0.0.0.0:*                   LISTEN
```

* 我们可以使用 “/FIN|TIME/” 来匹配 FIN 或者 TIME :

```bash
$ awk '$6 ~ /FIN|WAIT|ES/ {print NR,$1,$5,$6}' OFS="\t" netstat.txt

5	tcp	124.205.5.146:18245	TIME_WAIT
6	tcp	61.140.101.185:37538	FIN_WAIT2
7	tcp	110.194.134.189:1032	ESTABLISHED
8	tcp	123.169.124.111:49809	ESTABLISHED
9	tcp	116.234.127.77:11502	FIN_WAIT2
10	tcp	123.169.124.111:49829	ESTABLISHED
11	tcp	183.60.215.36:36970	TIME_WAIT
12	tcp	61.148.242.38:30901	ESTABLISHED
13	tcp	124.152.181.209:26825	FIN_WAIT1
14	tcp	110.194.134.189:4796	ESTABLISHED
15	tcp	183.60.212.163:51082	TIME_WAIT
17	tcp	123.169.124.111:49840	ESTABLISHED
18	tcp	117.136.20.85:50025	FIN_WAIT2
```

再来看看模式取反的例子：

```bash
$ awk '$6 !~ /FIN|WAIT/ {print NR,$4,$6}' netstat.txt

1	Local-Address	State
2	0.0.0.0:3306	LISTEN
3	0.0.0.0:80	LISTEN
4	127.0.0.1:9000	LISTEN
7	coolshell.cn:80	ESTABLISHED
8	coolshell.cn:80	ESTABLISHED
10	coolshell.cn:80	ESTABLISHED
12	coolshell.cn:80	ESTABLISHED
14	coolshell.cn:80	ESTABLISHED
16	coolshell.cn:80	LAST_ACK
17	coolshell.cn:80	ESTABLISHED
19	:::22
```

### 3.1 折分文件

awk拆分文件很简单，使用重定向就好了。下面这个例子，是按第6例分隔文件，相当的简单（其中的NR!=1表示不处理表头）。

```bash
$ ls
netstat.txt test.data
$ awk 'NR!=1{print > $6}' netstat.txt
ESTABLISHED	FIN_WAIT1	FIN_WAIT2	LAST_ACK	LISTEN		TIME_WAIT	netstat.txt	test.data

$ cat ESTABLISHED
tcp        0      0 coolshell.cn:80        110.194.134.189:1032        ESTABLISHED
tcp        0      0 coolshell.cn:80        123.169.124.111:49809       ESTABLISHED
tcp        0      0 coolshell.cn:80        123.169.124.111:49829       ESTABLISHED
tcp        0   4166 coolshell.cn:80        61.148.242.38:30901         ESTABLISHED
tcp        0      0 coolshell.cn:80        110.194.134.189:4796        ESTABLISHED
tcp        0      0 coolshell.cn:80        123.169.124.111:49840       ESTABLISHED

$ cat FIN_WAIT1
tcp        0      1 coolshell.cn:80        124.152.181.209:26825       FIN_WAIT1

$ cat FIN_WAIT2
tcp        0      0 coolshell.cn:80        61.140.101.185:37538        FIN_WAIT2
tcp        0      0 coolshell.cn:80        116.234.127.77:11502        FIN_WAIT2
tcp        0      0 coolshell.cn:80        117.136.20.85:50025         FIN_WAIT2

$ cat LAST_ACK
tcp        0      1 coolshell.cn:80        208.115.113.92:50601        LAST_ACK

cat LISTEN
tcp        0      0 0.0.0.0:3306           0.0.0.0:*                   LISTEN
tcp        0      0 0.0.0.0:80             0.0.0.0:*                   LISTEN
tcp        0      0 127.0.0.1:9000         0.0.0.0:*                   LISTEN
tcp        0      0 :::22                  :::*                        LISTEN
$ cat TIME_WAIT
tcp        0      0 coolshell.cn:80        124.205.5.146:18245         TIME_WAIT
tcp        0      0 coolshell.cn:80        183.60.215.36:36970         TIME_WAIT
tcp        0      0 coolshell.cn:80        183.60.212.163:51082        TIME_WAIT
```

* 你也可以把指定的列输出到文件：

```bash
$ awk 'NR!=1{print $4,$5 > $6}' netstat.txt
$ cat FIN_WAIT1
coolshell.cn:80 124.152.181.209:26825
```

* 再复杂一点：（注意其中的if-else-if语句，可见awk其实是个脚本解释器）

```bash
$ awk 'NR!=1{if($6 ~ /TIME|ESTABLISHED/) print > "1.txt";
else if($6 ~ /LISTEN/) print > "2.txt";
else print > "3.txt" }' netstat.txt
$ ls
1.txt		2.txt		3.txt
```

## 4. 统计

下面的命令计算所有的C文件，CPP文件和H文件的文件大小总和。

```bash
$ ls -l *.data *.txt | awk '{sum+=$5} END {print sum}'
3785
```

* 我们再来看一个统计各个connection状态的用法：（我们可以看到一些编程的影子了，大家都是程序员我就不解释了。注意其中的数组的用法）

```bash
awk 'NR!=1{a[$6]++;} END {for (i in a) print i ", " a[i];}' netstat.txt
, 1
LISTEN, 3
LAST_ACK, 1
FIN_WAIT1, 1
FIN_WAIT2, 3
TIME_WAIT, 3
ESTABLISHED, 6
```

* 再来看看统计每个用户的进程的占了多少内存（注：sum的RSS那一列）

```bash
ps aux | awk 'NR!=1{a[$1]+=$6;} END { for(i in a) print i ", " a[i]"KB";}'
```

## 5. awk 脚本

在上面我们可以看到一个END关键字。END的意思是“处理完所有的行的标识”，即然说到了END就有必要介绍一下BEGIN，这两个关键字意味着执行前和执行后的意思，语法如下：

* BEGIN{ 这里面放的是执行前的语句 }
* END {这里面放的是处理完所有的行后要执行的语句 }
* {这里面放的是处理每一行时要执行的语句}

为了说清楚这个事，我们来看看下面的示例：

假设有这么一个文件（学生成绩表）：

## 环境变量

即然说到了脚本，我们来看看怎么和环境变量交互：（使用-v参数和ENVIRON，使用ENVIRON的环境变量需要export）

```bash
$ x=5
$ y=10
$ export y
$ echo $x $y
5 10

$ awk -v val=$x '{print $1, $2, $3, $4+val, $5+ENVIRON["y"]}' OFS="\t" score.txt
Marry	2143	78	89	87
Jack	2321	66	83	55
Tom	2122	48	82	81
Mike	2537	87	102	105
Bob	2415	40	62	72
```

## 6. 几个花活

最后，我们再来看几个小例子：

### 6.1 从file文件中找出长度大于80的行

```bash
$ awk 'length>80' netstat.txt
tcp        0      0 coolshell.cn:80        110.194.134.189:1032        ESTABLISHED
tcp        0      0 coolshell.cn:80        123.169.124.111:49809       ESTABLISHED
tcp        0      0 coolshell.cn:80        123.169.124.111:49829       ESTABLISHED
tcp        0   4166 coolshell.cn:80        61.148.242.38:30901         ESTABLISHED
tcp        0      0 coolshell.cn:80        110.194.134.189:4796        ESTABLISHED
tcp        0      0 coolshell.cn:80        123.169.124.111:49840       ESTABLISHED
```

### 6.2 按连接数查看客户端IP

```bash
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr
575 0
   1 sockets
   1 c16721d47ee25a1b
   1 c16721d47e646ef3
   1 c16721d46acb0d83
   1 c16721d46acb0c8b
   1 c16721d46aca8f73
   1 c16721d46ac9f1db
```

### 6.3 打印九九乘法表

```bash
seq 9 | sed 'H;g' | awk -v RS='' '{for(i=1;i<=NF;i++)printf("%dx%d=%d%s", i, NR, i*NR, i==NR?"\n":"\t")}'
1x1=1
1x2=2	2x2=4
1x3=3	2x3=6	3x3=9
1x4=4	2x4=8	3x4=12	4x4=16
1x5=5	2x5=10	3x5=15	4x5=20	5x5=25
1x6=6	2x6=12	3x6=18	4x6=24	5x6=30	6x6=36
1x7=7	2x7=14	3x7=21	4x7=28	5x7=35	6x7=42	7x7=49
1x8=8	2x8=16	3x8=24	4x8=32	5x8=40	6x8=48	7x8=56	8x8=64
1x9=9	2x9=18	3x9=27	4x9=36	5x9=45	6x9=54	7x9=63	8x9=72	9x9=81
```