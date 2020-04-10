# 起因

额。。。一些特殊原因需要试探局域网下的几台主机，然后找不到以前存的Kali虚拟机镜像了。。就只好再配置一个。。



## 安装

安装过程没啥说的略。默认ssh不会启动，`service ssh start` 启动ssh就行了。为了方便使用打开root的远程连接权限。打开`/etc/ssh/sshd_config` 修改`PasswordAuthentication yes`和`PermitRootLogin yes`然后重启ssh服务

- 虚拟机用桥接，方便另一台笔记本访问
- 就算是Kali 虚拟硬盘也给30G吧。。。。

换源！安装常用软件！每次必做的事

```shell
# /etc/apt/sources.list
#中科大kali源
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
     
#浙江大学 Kali源
deb http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
deb-src http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
     
#官方Kali源
#deb http://http.kali.org/kali kali-rolling main non-free contrib
#deb-src http://http.kali.org/kali kali-rolling main non-free contrib 
#清华大学kali源
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
deb-src http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
```

- tmux htop 必须用的

## 配置Globalprotect

北邮的嘛.....，Gp软件有Mac都没有Linux...不知为何。不过协议是通用的，用openconnect可以替代。

```shell
apt-get install openconnect*
apt-get install vnc
```

再把保存的脚本文件扔到一个你找的到的地方，如`/etc/vpnc/vpnc_script`后，给文件可执行权限后执行`openconnect --protocol=gp --script=/etc/vpnc/vpnc_script vpn.bupt.edu.cn`即可

## 配置SSR

https://www.jianshu.com/p/7923ac1c378c?tdsourcetag=s_pctim_aiomsg

- 客户端安装

```
apt install -y python-pip
pip install --upgrade pip
pip install shadowsocks


~/.pip/pip.conf写入
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```

- 配置

```
echo { >~/ss.json
echo '"server" : "38.39.232.48",' >>~/ss.json
echo '"server_port" : 10001,' >>~/ss.json
echo '"password" : "246544",' >>~/ss.json
echo '"method" : "aes-256-cfb"' >>~/ss.json
echo } >>~/ss.json
```

- 运行

```
## 解决两个小bug
sed -i "s/libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)/libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)/g" /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py
sed -i "s/libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx)/libcrypto.EVP_CIPHER_CTX_reset(self._ctx)/g" /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py



root@kali:~# sslocal -c ~/ss.json
INFO: loading config from /root/ss.json
2020-04-11 00:21:30 INFO     loading libcrypto from libcrypto.so.1.1
2020-04-11 00:21:30 INFO     starting local at 127.0.0.1:1080

```

polipo可以用作全局代理

https://www.u22e.com/2539.html

这个网站说的很清楚，如果安不上的可以源码安装，注意自己写service文件的时候，ExecStart分别是执行文件路径和配置路径！

```
[Unit]
Description=polipo web proxy
After=network.target

[Service]
Type=simple
WorkingDirectory=/tmp
User=root
Group=root
ExecStart=/usr/bin/polipo -c /opt/polipo/config
Restart=always
SyslogIdentifier=Polipo

[Install]
WantedBy=multi-user.target
export http_proxy=http://127.0.0.1:8123
export https_proxy=http://127.0.0.1:8123
```

设置完成后，可以如此访问

- 加入代理前缀`http_proxy=http://127.0.0.1:8123`和`https_proxy=http://127.0.0.1:8123`
- 也可以直接export让当前命令行全走代理

```
root@kali:~# curl www.google.com | wc
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13961    0 13961    0     0   3260      0 --:--:--  0:00:04 --:--:--  3260
     10     384   13961
root@kali:~# 

```

## 铸神之眼nmap

花精力又是北邮vpn又是ssr的只是为了这一步