# Linux修改time_wait不起作用

博主为了在本地测试服务器，决定修改一下`time_wait`的时长（不知道这个东西的需要复习下tcp四次挥手），随意地在网上这么一搜大部分博客给出类似如下答案：

```
1.修改/etc/sysctl.conf文件中的若干参数
net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout = 5 修改系统默认的 TIMEOUT 时间

2.执行sysctl -p命令，来激活上面的设置永久生效
```

首先修改完后执行`sysctl -p`命令，可能会出现第一个问题，提示文件`net.ipv4.tcp_tw_recycle`不存在，这个是因为在linux内核`4.12`后这个参数已经被移除了 ref：https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=4396e46187ca5070219b81773c4e65088dac50cc

其次就是，修改后`time_wait`的时长还是2MSL即60秒，**修改根本不起作用**，用`ss -s`命令查看还是一堆的`time_wait`链接。这时我就觉得那些文章是不是又在水水水，于是就直接google了这个问题，得到如下的解释：

```
参数net.ipv4.tcp_fin_timeout实际上修改的是`FIN_WAIT_2`的超时时间
ref：https://www.linuxquestions.org/questions/linux-networking-3/how-to-really-decrease-time_wait-880134/
```

所以指望修改`tcp_fin_timeout`来修改`time_wait`的时长可以洗洗睡了，Linux也没有提供接口来修改这个值。

