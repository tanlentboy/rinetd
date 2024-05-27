在Linux系统中大多数情况选择用iptables来实现端口转发，iptables虽然强大，但配置不便，而且新手容易出错。在此分享另一个TCP/UDP端口转发工具rinetd，rinetd体积小巧，配置也很简单。

安装rinetd
这篇文章以Ubuntu 22.04为例，复制下面的命令输入，一行一个：

#安装依赖
``` 
sudo apt update
sudo apt install gcc g++ make
```
#下载rinetd
``` 
wget https://github.com/samhocevar/rinetd/releases/download/v0.73/rinetd-0.73.tar.gz
```
#解压
```
tar -zxvf rinetd-0.73.tar.gz
```
#进入目录
```
cd rinetd-0.73
```
#编译安装
```
./bootstrap
./configure 
make && make install
```
安装后，可以输入rinetd -v查看当前版本。
```
[root@kryptcn2 rinetd-0.73]# rinetd -v
rinetd 0.70
```
随着时间推移，上面下载地址不一定是最新的，大家可前往Github：https://github.com/samhocevar/rinetd/releases下载最新版本。

设置TCP端口转发
#新建rinetd配置文件
```
vi /etc/rinetd.conf
```
#填写如下内容
```
0.0.0.0 2018 103.74.192.160 2019
```
#启动rinetd
```
rinetd -c /etc/rinetd.conf
```

rinetd配置文件的格式如下:
```
0.0.0.0：源IP
2018：源端口
103.74.192.160：目标IP
2019：目标端口
```
上面配置的意思是将本地2018端口转发到103.74.192.160的2019端口，启动后可以输入netstat -apn|grep 'rinetd'查看是否运行正常，注意还需要在自己服务器防火墙放行对应的源端口，否则无法正常使用用。

从0.70版本开始rinetd已经支持UDP转发，写法如下：
```
127.0.0.1   8000/udp  192.168.1.2     8000/udp
```
创建systemd服务
为了方便管理，我们可以为rinetd编写一个systemd服务，有兴趣的同学可参考《Linux系统编写Systemd Service实践》，直接复制下面的内容即可：
```
#创建rinetd服务
sudo nano /etc/systemd/system/rinetd.service
```
复制下面的内容进行保存：
```
[Unit]
Description=rinetd
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/sbin/rinetd -c /etc/rinetd.conf

[Install]
WantedBy=multi-user.target

```
保存文件后，需要重新加载Systemd以识别新的服务文件。使用以下命令：
```
sudo systemctl daemon-reload
```
使其生效，然后就可以使用下面的命令来管理rinetd了。
```
#启动rinetd
systemctl start rinetd
#设置开机启动
systemctl enable rinetd
#停止rinetd
systemctl stop rinetd
#重启
systemctl restart rinetd
```
