# Linux常用命令

## 安装redis
```linux
yum install redis // 检查是否有redis yum 源
yum install epel-release //下载fedora的epel仓库
yum install redis
systemctl enable redis // 设为开机启动
systemctl start redis // 启动
```

## 查看内核版本：
cat /proc/version

## 查看系统版本信息：
cat /etc/issue

## 查看外部是否可以访问端口：
talnet ip 端口

cat /etc/sysconfig/iptables

netstat -nupl (UDP类型的端口)

## netstat命令
netstat -ntpl (TCP类型的端口)
- a 表示所有
- n表示不查询dns
- t表示tcp协议
- u表示udp协议
- p表示查询占用的程序
- l表示查询正在监听的程序
 
netstat -nuplf|grep 3306   //这个表示查找处于监听状态的，端口号为3306的进程


## df 查看分区分盘
df -h


## top 查看系统运行状态：
top [-d number] | top [-bnp]
- d 指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。 
- p 通过指定监控进程ID来仅仅监控某个进程的状态。 
- q 该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。 
- S 指定累计模式 
- s 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。 
- i 使top不显示任何闲置或者僵死进程。 
- c 显示整个命令行而不只是显示命令名 

**交互命令**：
- Ctrl+L 擦除并且重写屏幕。 
- h或者? 显示帮助画面，给出一些简短的命令总结说明。 
- k       终止一个进程。系统将提示用户输入需要终止的进程PID，以及需要发送给该进程什么样的信号。一般的终止进程可以使用15信号；如果不能正常结束那就使用信号9强制结束该进程。默认值是信号15。在安全模式中此命令被屏蔽。 
- i 忽略闲置和僵死进程。这是一个开关式命令。 
- q 退出程序。 
- r 重新安排一个进程的优先级别。系统提示用户输入需要改变的进程PID以及需要设置的进程优先级值。输入一个正值将使优先级降低，反之则可以使该进程拥有更高的优先权。默认值是10。 
- S 切换到累计模式。 
- s 改变两次刷新之间的延迟时间。系统将提示用户输入新的时间，单位为s。如果有小数，就换算成m s。输入0值则系统将不断刷新，默认值是5 s。需要注意的是如果设置太小的时间，很可能会引起不断刷新，从而根本来不及看清显示的情况，而且系统负载也会大大增加。 
- f或者F 从当前显示中添加或者删除项目。 
- o或者O 改变显示项目的顺序。 
- l 切换显示平均负载和启动时间信息。 
- m 切换显示内存信息。 
- t 切换显示进程和CPU状态信息。 
- c 切换显示命令名称和完整命令行。 
- M 根据驻留内存大小进行排序。 
- P 根据CPU使用百分比大小进行排序。 
- T 根据时间/累计时间进行排序。 
- W 将当前设置写入~/.toprc文件中。这是写top配置文件的推荐方法。


## free 显示内存使用状态：
free
-  -b  以Byte为单位显示内存使用情况。 
-  -k  以KB为单位显示内存使用情况。 
-  -m  以MB为单位显示内存使用情况。 
-  -o  不显示缓冲区调节列。 
-  -s<间隔秒数>  持续观察内存使用状况。 
-  -t  显示内存总和列。 
-  -V  显示版本信息


## 查看CPU内存占用情况：
ps -aux |grep pid

## centos7防火墙相关：
```linux
开放端口：
firewall-cmd --list-ports
状态：
firewall-cmd --state
停止：
systemctl stop firewalld.service
systemctl start firewalld.service
禁止开机启动：
systemctl disable firewalld.service
开机启动：
systemctl enable iptables.service
启动：
systemctl start firewalld.service
重启：
firewall-cmd --reload
永久开启端口重启生效：
firewall-cmd --zone=public --add-port=80/tcp --permanent
查看是否有该端口：
netstat -lntp|grep 7777
```


## 安装rz sz工具包：
yum -y install lrzsz

## 文件上传下载：
```linux
rz
批量上传（CTRL C取消）：
rz -be
sz
批量下载文件后缀为zip的文件（CTRL C取消）：
sz -a *.zip
二进制下载文件：
sz -be source.zip 
```

## 压缩到文件夹：
zip source.zip *.c


## 查找并输出结果到文件：
find . -name "*.zip" -print >  info.txt

## 查看服务安装相关文件：
whereis nginx

## 服务配置，开机启动或停止服务：
nginx可以换成其他服务，mysql，mq等
```linux
$ sudo systemctl enable nginx # 设置开机启动 
$ sudo service nginx start # 启动 nginx 服务 
$ sudo service nginx stop # 停止 nginx 服务 
$ sudo service nginx restart # 重启 nginx 服务 
$ sudo service nginx reload # 重新加载配置，一般是在修改过 nginx 配置文件时使用。
```


## >/dev/null 2>&1 
默认情况下，总是有三个文件处于打开状态，标准输入(键盘输入)、标准输出（输出到屏幕）、标准错误（也是输出到屏幕），它们分别对应的文件描述符是0，1，2 。那么我们来看看下面的几种重定向方法的区别：

\>/dev/null 2>&1 

实际上，应该等同于这样： 1>/dev/null 2>/dev/null ，默认情况下就是1，标准输出，所以一般都省略。 而&符号，后面接的是必须的文件描述符。不能写成2>1，这样就成了标准错误重定向到文件名为1的文件中了，而不是重定向标准错误到标准输出中。所以这里就是：标准输出重定向到了/dev/null，而标准错误又重定向到了标准输出，所以就成了标准输出和标准错误都重定向到了/dev/null

2>&1 >/dev/null 

咋一看，这个跟上面那个有啥区别呢，不也是标准错误重定向到标准输出，而标准输出重定向到/dev/null么？ 最后不应该都重定向/dev/null么？ 我是这么理解的！一条指令同一时刻要么产生标准错误，要么产生标准输出。 当产出标准错误的时候，因这个标准错误重定向到了标准输出，而标准输出是输出到屏幕。这个时候标准输出还没有被重定向到/dev/null，于是在屏幕上打印了。当产生标准输出时，那么它就不是标准错误，2>&1无效，于是标准输出重定向dev/null，不打印到屏幕。所以最终结果将是：标准错误打印到屏幕，而标准输出不打印到屏幕。
