---
title: Linux-shell脚本
date: 03-01-2021
categories: 
- linux
tags: 
- note
---

## 一、系统自带的shell脚本



### 1.作用
1. 用于linux系统的启动过程
2. linux的命令



### 2.linux的启动过程
1. BIOS引导-基本的输入输出系统 

	选择需要引导的内容：网络 or 硬盘 or 光盘
 
	系统启动时，按F2，可以选择引导介质

2. MBR-硬盘是否可引导

	dd if=/dev/sda of=mbr.bin bs=446 count=1 //查看硬盘的主引导记录，假设sda是要引导的硬盘，blocksize大小是446字节，用1块 

	hexdump -C mbr.bin //mbr.bin没有文件系统，无法用cat查看，hexdump用十六进制的方式查，-C表示能显示成字符的就显示成字符，BIOS通过读取该文件，判断是否可引导

3. BootLoader（grub）：启动和引导内核的工具

	grub文件的位置：/root/grub2，根据grub的配置文件选择内核，将权限交给内核继续引导

	grub2-editenv list //查看默认引导的内核版本信息

	uname -r //查看当前使用的内核

4. kernel：内核启动

	内核驱动硬件，初始化环境

5. systemd：

6. 系统初始化

7. shell工作



## 二、shell脚本格式



### 1.如何将cd/ls两条命令组合成一行
	- cd /tmp ; ls


	
### 2.shell脚本写完后，记得赋予脚本rx权限



### 3.shell脚本后缀：.sh



### 4.运行shell脚本：bash 1.sh



### 5.脚本开始写个声明：默认用bash运行该脚本
```shell
#!/bin/bash
# 上一行被叫做 Sha-Bang
```
这一行有什么作用呢？ 
1. 如果是bash 1.sh 运行脚本，那么这一行就被认为是注释，被忽略
2. 如何是./1.sh运行脚本，那么这一行就被解读为：现在要用bin下的bash来运行当前的脚本



### 6.注释以#开头



## 三、脚本运行的不同方式



### 1.不同的执行方式
```shell
#!/bin/bash
cd /tmp
pwd
```
接下来以以下四种方式运行上述脚本
1. bash ./filename.sh //在当前终端产生一个bash的进程，filename.sh就会成为该bash的子进程，不用为shell脚本赋予执行权限，就可以运行；运行完成后，目录并未发生变化
2. ./filename.sh //与上一条类似，但是必须为shell脚本赋予执行权限
3. source ./filename.sh //运行完成后，目录变为/tmp
4. . ./filename.sh //与上一条类似



### 2.由上一节得出结论：内建命令与外部命令的区别
1. 内建命令不需要创建子进程
2. 内建命令对当前shell生效



## 四、shell脚本语法



### 管道与重定向
1. 管道：｜
将｜左边命令的输出作为｜右边命令的输入
```shell
ls -l | more #将ls -l的执行结果传递给more
```
2. 重定向：>
一个进程在运行的时候，默认会打开【标准输入0/标准输出1/错误输出2】三个文件描述符
	- 输入重定向: <
```shell
read var < /path/afile
```
	- 输出重定向：>(重写) / >>(追加) / 2>(错误重定向) / &>(无论输出正确错误，都输出)
```shell
echo 123 > /path/file
```
	- 将输入输出重定向组合在一起使用
```shell
#!bin/bash
cat > /tmp/bfile <<EOF #该脚本要以EOF结尾
echo “Hello World”
EOF
```
运行上述脚本，再查看/tmp/bfile，可以看到该文件中是 echo “Hello World”



### 变量
1. 定义变量
变量名称：只能包含字母数字下划线；不能以数字开头
2. 变量赋值
赋值符号=的左右不能出现空格
```shell
# 变量名=变量值
a=123
# 用let赋值，可以在=右侧进行计算
let a=1+1
# 将命令赋值给变量，
a=ls
# 将命令执行结果赋值给变量$() or ``
a=$(ls -l /tmp)
a=`ls /tmp`
# 变量值中包含空格或特殊字符时，用双引号括起来
a=“Hello World”
```
3. 变量的引用
	- ${变量名} 
	- $变量名
	- 什么时候可以省略{}
```shell
A=“hello world”
echo ${A} # 语法正确
echo $A # 语法正确
echo ${A}otherstrings # 语法正确：输出hello worldotherstrings
echo $Aotherstrings # 语法错误，echo会认为变量名时Aotherstrings
```
4. 变量的作用范围
默认只在自己的shell范围中，不会作用到父进程/子进程/平行进程
如何希望shell脚本里的变量对当前终端生效，用source或. 执行该shell脚本
	- 如何让子进程得到父进程的变量
```shell
var_demo=“hello world”
export $var_demo #接下来，在该终端下的子进程都可以获得该变量
# 或者：export var_demo=“hello world”
# 需要var_demo这个变量时，用以下方式进行删除
unset var_demo
```



### 环境变量/预定义变量/位置变量
1. 什么是环境变量？

	将变量保存在系统中，让每个shell都可以读取到该变量的值
2. 如何查看已有的环境变量
```shell
env | more
```
3. 如何查看某一个环境变量的值
```shell
echo ${SHELL}
```
4. $PATH 变量

	该变量中记录着命令搜索路径，平时执行的命令存放的位置
5. 如何将/tmp目录添加到$PATH中
```shell
PATH=${PATH}:/tmp #只对当前终端和子进程生效，重新打开一个终端，不生效
```
6. $PS1变量

	让终端显示信息更友好，可以google或baidu一下，如何修改
7. 可以用set命令查看到更多变量，包括预定义变量和位置变量
8. 常用的预定义变量：`$? $$ $0`
	- `$?`:上一条命令是否正确执行，正确则为0，不正确非0
	- `$$`:当前进程的pid
	- `$0`:当前进程的名称
9. 位置变量
	- 有什么用？当一个shell脚本执行的时候，有输入时，用于标识输入变量
```shell
#!/bin/bash
# $1 $2... $9 ${10}
post1=$1
post2=$2
echo $post1
echo $post2
```
	- 为上述脚本赋予执行权限，记上述脚本为post.sh
执行post.sh：
```shell
./post.sh -a -b #那么-a就会被赋值给post1，-b就会被赋值给post2
```
	- 当$1为空值时如何处理？
```shell
${1}_ #表示如何$读取为空，那么将_赋值给$1,但是如果$1被赋值了，那么这个值后面就会呆着1个下划线
${1-_} #解决上一条命令的问题
```



### 环境变量的配置文件
1. 4个配置文件的路径
	- /etc/profile #所有用户通用
	- /etc/bashrc #所有用户通用
	- ~/.bash_profile #当前用户特有
	- ~/.bashrc #当前用户特有
	- /etc/profile.d/ #这是一个【配置文件的】目录

2. profile/bashrc这两个配置文件有什么区别？-login与nologin的区别

	登录一个用户时，分为login shell 和nologin shell 

	su后面有-时，表示login shell，四个配置文件都会被执行 

	su后面没有-时，表示nologin shell，bashrc的配置文件会被执行

3. 如何在配置文件中添加环境变量？
```shell
# 在上述任意一个配置文件的最后加上以下这行，就可以添加环境变量
export PATH=$PATH:/new/path
```
4. 切换或登录用户时，尽量用su - user（加上-），这样配置文件可以加载完全
5. 配置文件中新加的环境变量，不会立即生效；
 
	立即生效的方法1:关掉终端，再打开 

	方法2:source /etc/bashrc //用source运行新添加的配置文件



### 数组
```shell
# 定义数组，元素用空格隔开
alist=(abc def ghi)
# 显示数组的所有元素
echo ${alist[@]}
# 显示数组的元素个数
echo ${#alist[@]}
# 显示第1个元素
echo ${alist[0]}
```



### 转义与引用
1. 特殊字符
- #注释
- ; f分号，连接两条较短命令或语句
- \转义符号:
	- \n\t\r
	- \$ \” \\
- 引号
	- “双引号，echo ”$a” #输出变量a的值
	- ‘单引号 ，`echo ’$a’ #输出$a`
	- `反引号，
	
	
	
### 运算符
1. 赋值=
- 用于算数赋值和字符串赋值
- unset 可以取消对变量的赋值
2. 运算
- +-*/ ** %
- expr(只接受整数)
	- expr 4+5
```shell
((a=10))
((a=1+1))
((a++))
echo $((10+20))
expr 1 + 1 #注意数字之间需要有空格
num1=`expr 1 + 1`
echo ${num1}
```



### 特殊符号
1. 引号
	- ‘完全引用
	- “不完全引用
	- ` 执行命令
2. 括号
	- () (()) $()
		- 单独使用一个圆括号，会产生一个子shell
		- 数组的初始化
		- ((1+1))
		- cmd1=$(ls) # 取命令的执行结果
	- [] [[]]
		- []测试命令 [5 -gt 4]
		- [[5 > 4]]
	- <> 大小比较/重定向
	- {} 
		- echo {0..9} #输出0-9这个范围的数字
3. 运算符和逻辑符号
	- +-*/%
	- \> < = 比较运算符
	- && || !
```shell
(( 2>1))
echo $?
((5>4 && 7>8))
echo $?
((! 3 > 1))
echo $?
```
4. 其他
	- #
	- ;
	- : //相当于pass
	- .
	- ./
	- ~ //家目录
	- cd - //进入上一次访问的目录
	- *
	- ？
	- $
	- &


	
### 测试和判断
1. 退出程序命令
```shell
#!/bin/bash
pwd
exit #如果以上命令出错，exit会返回非零值，显示在终端
```
2. test命令-用$?判断
man test //可以查看到各种逻辑比较相关的语法
```shell
# 判断是否是文件
test -f /etc/passwd
echo $? # 是文件，返回0
# 判断是否是目录
[ -d /etc/ ]
echo $? # 是目录，返回0
# 判断文件是否存在
[-e /etc/passwd ]
# 数字判断与比较
[ 5 -gt 5]
# 字符串比较
[ “abc” = “abc”]
echo $?
```



### if语句
1. 语法
- if \[ 测试条件成立 \] 或 命令返回值是否为0
- then 执行相应命令
- fi 结束
```shell
if [ $UID = 0 ] then
echo $?
fi
if [ $USER = root ]then
echo $?
fi
if [ $UID = 0 ]
then
	echo “ root user ”
fi
if pwd; then echo “pwd running”; fi
```
2. if else语法
- if \[测试条件成立\] 或 命令返回值是否为0 
- then 执行相应命令 
- else 测试条件不成立，执行相应命令 
- fi 结束
```shell
#!/bin/bash
if [ $USER = root ];
then 
	echo “root user”
	echo $UID
else
	echo “other user”
	echo $UID
fi
```
3. if elif语法<br>
if ;then ;elif ;then else ;fi

4. 嵌套if
	- if \[ 测试条件成立 \] 
	- then 执行相应命令 
	-     if \[测试条件成立\]
	-     then 执行相应命令
	-     fi
	- fi
```shell
if [$UID = 0]; then
	if [ -x /tmp/10.sh ]; then
		/tmp/10.sh 
	fi 
fi
```



### case分支
```shell
#!/bin/bash
# case demo
case “$1” in
	“start”|”START”)
	echo $0 starting......
	;;
	“stop”|”STOP”)
	echo $0 stopping......
	;;
	“restart”|”reload”)
	echo $0 restarting.....
	;;
	*)
	echo “Usage: $0 {start|stop|restart|reload}”
	;;
esac
```



### for语句
for 参数 in 列表<br>
do 执行的命令<br>
done 封闭一个循环<br>
```shell
for i in {1..9}
do
	echo “Hello”
	echo $i
done
# 批量文件改名
touch afile.txt bfile.txt cfile.txt
for filename in `ls *.txt`
do
	mv $filename $(basename $filename .txt).md
done
```
C语言风格的for命令<br>
for((变量初始化;循环判断条件;变量变化))<br>
do<br>
&nbsp;&nbsp;&nbsp;&nbsp;循环执行的命令<br>
done<br>
```shell
for (( i=1 ; i<=10 ; i++ ))
do
	echo $i
done
```



### while语句
while test测试是否成立<br>
do<br>
&nbsp;&nbsp;&nbsp;&nbsp;命令<br>
done<br>
```shell
# while循环
a=1
while [ $a -lt 10 ] 
do
	echo $a;
	((a++));
done
# 创建死循环
while :
do
	echo 123
done
# util循环,当条件为假时，进入循环
util [ 5 -lt 4]
do
	echo 123
done 
```



### 循环嵌套/break/continue
```shell
# 循环嵌套
for filename in /etc/profile.d/*.sh
do
	echo $filename
	if [ -x $filename ];
	then . $filename
	fi	
done
# break/continue
for num in {1..9}
do
	echo $num
	if [ $num -eq. 5 ];
	then break;
	fi;
done
```



### 用循环处理位置参数
1. 命令行参数可以用`$1 $2 $3..$n`表示
2. $0表示脚本名称
3. `$*和$@`代表所有位置参数
4. $# 代表位置参数的数量
```shell
# for循环处理位置参数
for param in $*
do
	if [ “$param” = “help” ]; then
		echo $param $param
	fi
done
# while循环处理位置参数
while [ $# -gt 1]
do
	echo $#
	if [ “$1” = “help”]; then
		echo $1 $1
	fi	
	shift #这个命令可以实现参数左移，删掉最左边一个参数
done 
```

### 自定义函数
```
# 函数定义
function fname(){
命令
}
# 函数执行
fname
```
```shell
function cdls(){
cd /var
ls
}
cdls
#取消cdls函数的定义
unset cdls
```

函数作用范围的变量

```local 变量名```

函数的参数

```$1 $2 $3```
```shell
# 定义带参数的函数
function cdls(){
cd $1
ls
}
# 执行带参数的函数
cdls /tmp
# 定义
function checkpid(){
	# 本地化变量i
	local i
	for i in $* ; do
		[ -d “/proc/$i” ] && return 0
	done
	# 没有进程存活，返回1
	return 1
}
```

如何执行定义在文件filename.sh中的函数checkpid

```
# filename.sh中定义的函数和变量在当前shell中生效
source filename.sh
checkpid 100
```

### 系统函数库
1. 系统自建函数库位置

/etc/init.d/functions目录下定义了常用的函数脚本文件

2. 如何使用系统自建的函数脚本文件

用 source 函数脚本文件 导入函数

### 脚本控制
1. 脚本优先级控制

cpu占用优先级：用nice/renice调整优先级

```
# 查看当前终端系统的一些限制
ulimit -a
```

```shell
# 实现一个fork炸弹，死循环创建子进程,cpu占满，不响应
func(){ func | func &}
func
```
内存占用优先级：

2. 捕获信号
- kill默认发送15号信号给应用程序
- ctrl+c发送2号信号给应用程序
- 9号信息不可阻塞

```shell
#!/bin/bash
# 信号演示
# 捕获信号
trap “echo sig 15” 15 #当接收到15号信号时，输出sig 15，忽略kill指令
trap “echo sig 2” 2 #忽略ctrl+c
echo $$ # 输出当前进程号
#如果没来得及捕获或捕获完，进入死循环
while :
do
	:
done
```

### 计划任务
1. 一次性计划任务 at

```shell
date
at 18:31
echo hello > hello.txt
# 再按ctrl+D 提交任务
# 查看计划任务
atq
```

2. 周期性计划任务

crontab命令

crontab -e //配置周期性计划任务

crontab -l //查看周期性计划任务

crontab -e 分钟 小时 日期 月份 星期 执行的命令 
```shell
crontab -e #进入类似vim的编辑界面
* * * * * /usr/bin/date >> /tmp/date.txt #每分钟运行一次
* * * * 5 /usr/bin/date >> /tmp/date.txt #每周五运行一次
* * * * 1,5 /usr/bin/date >> /tmp/date.txt #每周一和周五运行一次
* * * * 1-5 /usr/bin/date >> /tmp/date.txt #每周一至周五运行一次
```
查看计划任务有没有执行，可以查看/var/log/cron，这里会记录几月几日几时几分几秒运行了什么任务

每个用户有一个cron列表，记录在 /var/spool/cron/用户名 目录下

3. 计划任务加锁 flock

如果计划任务由于关机断网等因素不能按照预期时间运行，可以采用延期计划任务或给计划任务加锁

anacontab 延时计划任务

/etc/cron.d/0hourly

/etc/anacrontab

```shell
```
flock 锁文件

不希望脚本filename.sh被重复运行，
```
flock -xn “/tmp/f.lock” -c “filename.sh” # 再打开一个终端，运行该脚本，将无法运行 
```