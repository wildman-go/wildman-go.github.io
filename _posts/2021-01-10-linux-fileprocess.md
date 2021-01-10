---
title: Linux-文件处理
date: 10-01-2021
categories: 
- linux
tags: 
- note
---

## 一、正则表达式及文本搜索

### 1. 元字符
1. .匹配除换行符外的任意单个字符
2. *匹配任意一个跟在它前面的字符
3. []匹配方括号中的字符类中的任意一个字符
4. ^匹配开头
5. $匹配结尾
6. \转义后面的特殊字符
7. +匹配前面的正则表达式至少出现一次
8. ？匹配前面的正则表达式出现0次或1次
9. ｜匹配它前面或后面的正则表达式

### 2. grep
```
grep abc /tmp/123.sh #在/tmp/123.sh中查找包含123的行
grep pass.... /tmp/123.sh
grep pass....$ /tmp/123.sh
grep pass.* /tmp/123.sh
grep [Hh]ello /tmp/123.sh
grep “\.” /tmp/123.sh #记得加上双引号
```

## 二、find--文件查找
用途：用于文件查找

find 路径 查找条件
```
find /etc/passwd #只一对一匹配
find /etc/passwd*
find /etc -name passwd
find -regex .*wd$
find /etc -type f -regex .*wd #指定的文件类型，d目录，f普通文件
find /etc -atime 8 # 查找8小时以前被访问的文件
find /etc -mtime 8 # 查找8小时以前内容被更新的文件
find /etc -ctime 8 # 查找8小时以前文件被更新的文件
find /ect -user root # 查找归属root的文件
```

批量创建和删除文件
```
touch /tmp/{1..9}.txt
find /tmp/*txt -exec rm {} \
find /tmp/*txt｜rm -rf
```

对grep到的字符串做分割，并取其中一部分
```grep passwd /etc/ ｜cut -d “ ” -f 1 # 以空格作为分割，取第一部分```

## 三、sed
未完待续、、、、、、

## 四、awk














