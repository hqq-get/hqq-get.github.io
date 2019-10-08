---
title: linux-文件查找
top: false
cover: false
toc: true
mathjax: true
date: 2019-10-08 19:54:42
password:
summary:
tags:
categories:
---
### locate find

---

#### locate
- 安装

```
yum install mlocate -y
```

- 依赖于事先构建好的索引库

```
系统自动实现（周期性任务）
手动更新数据库（updatedb）
```
- 工作特性

```
1.查找速度快
2.模糊查找
3.非实时查找
```
- 使用方法

```
locate [OPTION]... PATTERN...
       -b, --basename
              Match only the base name against the specified patterns.  This is the opposite of --wholename.
       -c, --count
              Instead of writing file names on standard output, write the number of matching entries only.

```


#### find
- 实时查找工具，通过遍历指定起始路径下文件系统层级结构完成文件查找
用法

```
find [OPTIONS] [path] [查找条件] [处理动作]

查找起始路径: 指定具体搜索目标起始路径，默认为当前目录。
查找条件： 指定的查找标准，可以根据文件名，大小，类型，从属关系，权限等标准进行；默认为找出指定路径下的所有文件
处理动作： 对符合查找条件的文件做出的操作，例如，删除等操作。默认为输出至标准输出

```
- 根据文件名查找
```
-name "pattern"
-iname "pattern"  不区分大小写
pattern支持glob风格的通配符: *,?,[],[^]
```
- 根据属主属组查找
```
-user
-group
-uid
-gid
-nouser
-nogroup 

find /tmp -user root
find /tmp -group root
find /tmp -uid 3002

```


- 根据文件的类型查找

```
-type d   目录
-type f   文件
-type l   链接
-type b   块设备文件
-type c   字符设备文件
-type p   管道文件
-type s   套接字文件
```
- 组合查找

```
与: -a 默认组合逻辑，可以不加
或: -o 
非： -not 
find /tmp -nouser -a type f -ls
find /tmp -nouser -o type f -ls
find /tmp -not -type f -ls
find /tmp -not -user root -a -not -type f -ls 等同于 find /tmp -not \( -user root -o -type f \) -ls
```

- 根据文件的大小查找

```
-size [+|-] #UNIT
常用单位：K,M,G
#UNIT：(#-1,#]  不加加减号为精确匹配
-#UNIT: [0,#-1]
+#UNIT: [#,oo)
```
- 根据时间戳查找

```
以天为单位
 -atime [+|-]#
 -mtime [+|-]#
 -ctime [+|-]#
 
以分钟为单位
-amin [+|-]#
-mmin [+|-]#
-cmin [+|-]#
```
- 根据权限查找

```
-perm [/|-]mode
mode: 精确权限匹配
/mode: 任何一类用户(u,g,o)的任何一位权限(r,w,x)满足即符合条件
-mode: 
find ./ -perm /666 -ls
find ./ -perm 666 -ls
find ./ -perm /002 -ls 找其他用户有写权限的文件
find ./ -perm -222 -ls 所有用户都有写权限的文件
find ./ -not -perm -222 -ls 至少有一个用户没有写权限的文件
```

- 处理动作

```
 -print: 输出至标准输出，默认的动作 
 -ls: 相当于执行ls -l命令
 -delete: 删除
 -fls: 保存至文件
 -exec command {} \; 执行命令
```
注意:
find传递查找到的文件路径至后面的命令时，是先查找出所有符合条件的文件路径，并一次性传递给后面的命令。但是有些 命令不能接受过长的参数，此时命令执行会失败，用xargs可规避此问题

```
find . -maxdepth 1 -type d | xargs -n30 -t -r rm -rf 
```

#### xargs用法
xargs - build and execute command lines from standard input

useage:

```
 -i:  拿到标准输出结果，在本次命令中通过{}使用
 -t: 执行命令之前先打印
 -n, Use at most MAX-ARGS arguments per command line
 -r, --no-run-if-empty      If there are no arguments, run no command.
                            If this option is not given, COMMAND will be
                            run at least once.
```
e.g.

```
找出当前目录下超过30天的一级目录并删除
find . -maxdepth 1 -mtime +30 -type d | xargs -n30 -t -r rm -rf
```