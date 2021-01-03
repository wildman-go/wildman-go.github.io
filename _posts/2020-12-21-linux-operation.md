---
title: linux操作
date: 21-12-2020
categories: 
- linux
tags:
- note
---

## 一、第一次启动linux

#### 1、如何切换到字符终端
```
打开终端
init 3
进入 localhost login：
输入用户名
输入密码
回车
```

#### 2、如何切换用户
```
exit
回车
```

#### 3、有哪几个终端：
1. 图形终端
2. 命令行终端
3. 远程终端（SSH、VNC）

#### 4、Linux常用目录结构
- / 根目录
- /root root 用户的家目录
- /home/username 普通用户的家目录
- /etc 配置文件目录
- /bin 命令目录
- /sbin 管理命令目录
- /usr/bin /usr/sbin 系统预装的其他命令

#### 5、关机命令
init 0
回车

## 二、Linux系统的操作

#### 1、帮助命令：man/help/info
```
//man
man ls 
man man
man 7 man //获取第7章节的man命令帮助，每个章节有对应的分类，可以在第1章看到具体分类
man -a passwd //不确定是命令还是配置文件的帮助，用-a

//help
help cd //获取 内部命令 的帮助
ls —help //获取 外部命令 的帮助
type cd //查看cd命令是内部命令 or 外部命令

//info
info ls //info帮助 比help更详细
```

#### 2、pwd和ls
```
pwd

cd /
cd ./
cd ../
cd -  //该命令可以回到上一次所在的目录

ls 
 -l //长格式显示文件
 -a //显示隐藏文件
 -r //逆序显示
 -t //按照时间顺序显示
 -R //递归显示
 -h //以M显示文件大小
```

#### 3、创建和删除目录
 - mkdir
    创建已存在的文件夹，会提示文件已存在，创建失败
 ```
 mkdir /a  //在根目录创建一个文件夹a
 mkdir a  //在当前目录创建一个文件夹a
 mkdir a b c  //在当前目录创建3个文件夹：a b c 
 mkdir a b c -p //忽略已存在目录时报错
```
 - mkdir创建多级目录
 ```
 mkdir -p /a/b/c
 ```
 - 删除目录
```
 rmdir /a //只能删除非空目录
 rm -rf /a //递归删除目录a，并且不进行提示
```

#### 4、复制和移动目录
 - 复制目录
 ```
 cp -r /root/a /tmp  //将目录a 复制到tmp目录下
```
 - 复制文件
```
cp /filea /tmp //将根目录下的文件filea 复制到 tmp目录下
cp -p /filea /tmp //复制后保留文件的修改时间
cp -ap /filea /tmp //复制后保留文件的修改时间、权限、属主
```
 - 重命名文件
```
mv /filea /fileb
```
 - 移动文件
 ```
mv /filea /tmp
```
 - 移动并重命名移动后的文件
```
mv /filea /tmp/fileb
```
 - 移动目录
```
mv /dira /tmp //将根目录下的dira目录移动到tmp目录下
```
 - 通配符 * 匹配任意多个字符
 - 通配符 ? 匹配1个字符
 
#### 5、文本查看
 - cat
 - head
    - head -5 /demo //查看demo文件头5行内容
 - tail
    - -f //同步文本的更新
    - tail -3 /demo //查看demo文件尾部3行内容
 - wc
    - wc -l /demo //查看demo文件有多少行
 - more or less
 
#### 6、打包和压缩
 - tar cf /tmp/etc-backup.tar /etc //将etc目录打包为.tar文件
 - tar czf /tmp/etc-backup.tar.gz /etc //打包并压缩
 - c：打包
 - x：解包
 - f：指定操作类型为文件
 - tar xf /tmp/etc-backup.tar -C /root //将打包好的tar包解包到root目录下
 - tar xzf /tmp/etc-backup.tar.gz -C /root
 
#### 7、Vim的四种模式
1. 正常模式
    - i：进入插入模式
    - I：进入插入模式，光标位置在当前行的开头
    - a：进入插入模式，光标位置向后移动一个
    - A：进入插入模式，光标位置在当前行的结尾
    - o：进入插入模式，光标位置在当前行的下一行
    - O：进入插入模式，光标位置在当前行的上一行
    - esc：返回到正常模式
    - v：进入可视模式
    - :：进入命令模式
    - hjkl：h光标左移；l光标右移；j光标下移；k光标上移
    - 复制：将光标移动到待复制行，按yy；将光标移动到复制行，按p，复制成功
    - 复制多行：将光标移动到复制行，按3yy，就会复制当前行往下3行；~~~
    - 复制光标当前位置到结尾：按y$，就会复制至行结尾；~~~
    - 剪切命令:dd,d&，用法与yy一致
    - u：撤销
    - ctrl+r：反撤销
    - x：删除光标所在字符
    - r：按下r键，输入新字符，光标所在字符就会被替换
    - 光标移动到指定行：按11，再G，光标就会移动到第11行
    - g：光标移动到第一行
    - G：光标移动到最后一行
    - shift+^：光标移动到当前行开头
    - shift+$：光标移动到当前行结尾
2. 命令模式
    - :set nu：显示行号
    - :w //保存到当前文件
    - :w /root/a.txt //保存到指定文件
    - :q //退出
    - :!指定临时命令，执行完成后回车，再回到编辑器中
    - /x //再当前文件中查找x，按n进行查找下一个，N进行查找
3. 可视模式