---
date: 2019-01-31 00:23
status: public
title: linux使用的常见问题
---

# linux使用的常见问题

- 回车换行与换行
- 系统语言修改
- 回车出现^H
- Ctrl Z的误用
- curl 发送http请求
## 回车换行与换行 
[回车(CR)与换行(LF)， '\r'和'\n'的区别](https://www.cnblogs.com/me115/archive/2011/04/27/2030762.html)
- win: 回车换行 0d 0a
- Unix/Linux： 换行
- Mac OS： 回车
```
pris_syn@prisSyn:~$ cat test.txt 
012
345
pris_syn@prisSyn:~$ file test.txt 
test.txt: ASCII text
pris_syn@prisSyn:~$ od -t x1 test.txt 
0000000 30 31 32 0a 33 34 35 0a
0000010
pris_syn@prisSyn:~$ unix2dos test.txt 
unix2dos: 正在转换文件 test.txt 为DOS格式...
pris_syn@prisSyn:~$ od -t x1 test.txt 
0000000 30 31 32 0d 0a 33 34 35 0d 0a
0000012
pris_syn@prisSyn:~$ file test.txt 
test.txt: ASCII text, with CRLF line terminators
pris_syn@prisSyn:~$ dos2unix test.txt 
dos2unix: 正在转换文件 test.txt 为Unix格式...
pris_syn@prisSyn:~$ unix2mac test.txt 
unix2mac: 正在转换文件 test.txt 为Mac格式...
pris_syn@prisSyn:~$ file test.txt 
test.txt: ASCII text, with CR line terminators
pris_syn@prisSyn:~$ od -t x1 test.txt 
0000000 30 31 32 0d 33 34 35 0d
0000010
```
[ASCII,Unicode,utf8编码的区别](https://blog.csdn.net/Deft_MKJing/article/details/79460485)
## 系统语言修改
有时候Linux系统cat一个windows系统传上去的文件会乱码。原因是不同系统的编码问题。Linux 一般用utf-8编码，win使用gbk编码。二者在ASCII码部分没有区别，一个字节为零，后七位表示128个字符编码。拿中文为例，gbk用两个字节编码，首位是一来与ASCII码区分，而尴尬的是utf8是三个字节表示一个中文。
```
pris_syn@prisSyn:~$ echo $LANG
zh_CN.UTF-8
pris_syn@prisSyn:~$ echo "你好" | od -t x1
0000000 e4 bd a0 e5 a5 bd 0a
0000007
pris_syn@prisSyn:~$ echo "你好" | iconv -f utf8 -t gbk | od -t x1
0000000 c4 e3 ba c3 0a
0000005
```
一般的处理办法都是把文件通过iconv,或者从来源处换成utf8编码的文件，毕竟入乡随俗。我接触过几个地方的服务器，仅仅遇到一次，不知管理员出于何目的把系统编码改成了gb18030...搞得安装py库都报错。。。。当时我是这么弄得
```
# 查看语言设置
locale
# 打印出很多LC_的东西，那个大哥把LC_ALL设置成gb18030,我到现在仍然佩服....
# ubuntu
vim /etc/default/locale
# ret hat系列旧的
vim /etc/sysconfig/i18n
# centos 7
vim /etc/locale.conf
# 设置LANG
LANG="en_US.UTF-8"
# 使其立即生效
source  对应的配置文件 
```
为什么说佩服那个大哥呢，因为他的LC_系列我依稀记得是utf8的，他把LC_ALL改了，搞得我`export  LANG=en_US.UTF-8`都没用(改临时是不想都服务器的配置，但是后来知道一直是我们用就不客气了)。LANG是优先级很低的一个变量，它指定所有与locale有关的变量的默认值。但是LC_ALL可以管所有的locale(LC_ALL > LC_* >LANG)。所以这位无名大哥下手好狠...
## 回车出现^H
这个是碰到最多最多的了，基本就是bash正常，结果一运行程序就跪了。每次都改行律后来干脆Ctrl + Backspace....
```
# 查看行律设置
(py36) pris_syn@prisSyn:~$ stty -a
speed 38400 baud; rows 24; columns 108; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; 
# 发现erase没设置好
(py36) pris_syn@prisSyn:~$ stty erase ^H
(py36) pris_syn@prisSyn:~$ stty -a
speed 38400 baud; rows 24; columns 108; line = 0;
intr = ^C; quit = ^\; erase = ^H;
```
有时候vim下小键盘也会炸毛，这就没必要一个一个查改行律了，xshell下调整一下终端设置就好了。
其实有时候ls 中文目录乱码，最好看看终端模拟器的设置，不要去改服务器。

## Ctrl Z的误用
一个小故事，希望主人公看不到。一次她测试系统，运行完顺手一个Ctrl Z，我看到一个醒目的[14]  stopped.....
Ctrl Z一般是运行一个程序，发现很占用资源然后你现在想做点别的事，你就可以先将它挂起，弄好了再fg到前台，或者bg后台执行也可以，也可以用jobs查看一下挂起的进程。
这些其实都是行律在控制

- Ctrl-c Kill foreground process 
- Ctrl-z Suspend foreground process 
- Ctrl-d Terminate input, or exit shell 
- Ctrl-s Suspend output 
- Ctrl-q Resume output 
- Ctrl-l Clear screen

## curl 发送http请求
这个网上教程太多了...链接都懒得粘贴，写他的目的就是记录下怎么上校园网关...
```
curl --silent -d "user=账号&pass=密码&line=" "http://10.3.8.211/login"
```