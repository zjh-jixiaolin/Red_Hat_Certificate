# RHCSA8  试题解析

---

[TOC]



==**考试环境说明：**==

虚拟机 1： `node1.example.com`

虚拟机 2：`node2.example.com`

虚拟机 3：`classroom.example.com (考官机器)`



==**练习环境：**==

虚拟机1：`node1.example.com （node2 的题目也在 node1 完成即可）`

虚拟机2：`classroom.example.com`



## 1、 部署练习环境

1、安装一台虚拟机，为虚拟机`node1`添加 `2块 SATA 硬盘`，然后执行以下初始命令：

### 实例：

```cython
# 对指定网卡设备进行 **修改IP地址**
ifconfig ens160 172.25.0.11/24

# 更改 防火墙 默认区域为trussted（可信的）,默认放行所有连接请求。
firewall-cmd --set-default-zone=trusted

# 下载文件 并以 指定文件夹 命名
wget -O /usr/bin/rht_setup200 http://172.25.0.254/rht_setup200

# 赋予  文件的 执行权限（运行软件的权限）
chmod +x /usr/bin/rht_setup200

# 使用文件
rht_setup200
```

#### 解析：

##### 查看新添加磁盘

---

```cython
# 查看磁盘信息，黄色字体为新添加的硬盘。
ll /dev/sd*    # /dev/ 设备文件保存的位置
# ----------------------------------------------------
brw-rw----. 1 root disk 8,  0 Sep 26 17:34 /dev/sda
brw-rw----. 1 root disk 8, 16 Sep 26 17:34 /dev/sdb
```

##### 查看网卡和网络配置

---

```cython
# ifconfig 命令查看 （网卡配置 与 网络状态等信息） 
ifconfig
# -----------------------------------------------------
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
inet 192.168.17.130  netmask 255.255.255.0  broadcast 192.168.17.255
# -----------------------------------------------------
网卡设备(ens160)	UP:表示“接口已启动”    BROADCAST：表示“主机支持广播”

RUNNING：表示“接口在工作中”		MULTICAST：表示“主机支持多播”	MTU：1500（最大传输单元）

inet：网卡的IP地址 	netmask：网络掩码	boradcast：广播地址
```









## I、虚拟机1 执行的任务

### 1、配置网络设置

---

**试题概述:**

为虚拟机1配置以下网络参数：

主机名：`node1.example.com`

IP地址：`172.25.0.11`

子网掩码：`255.255.255.0`

网关:`172.25.0.254`

名称服务器:`172.25.254.254`



#### 实例：

nmcli就是NetworkManager的cli(命令行)

```cython
# modify(修改)网卡配置 (Wireed connection 1:网卡名称)
nmcli connection modify "Wireed connection 1" ipv4.method manual ipv4.addresses 172.25.0.11/24 ipv4.gateway 172.25.0.254 ipv4.dns 172.25.254.254 connection.autoconnect yes # 自动连接

#（激活连接，激活新网卡的配置）
nmcli con up "Wireed connection 1" 

# 永久修改主机名
hostnamectl set-hostname node1.example.com
```

##### 解析：

###### 修改 网卡配置 信息

---

```cython
# 命令语法
nmcli connection modify Con-Name（网卡名称） [+|-]setting.propertyvalue（设置属性值）

setting.property:

ipv4.method (manual(静态) | auto（动态）) | 方法（手动 、自动）
ipv4.addresse （IP地址）
ipv4.gateway （网关）
ipv4.dns1 （DNS）

connection.autoconnect yes | 	# 设置网卡自启动，实际修改的网卡配置文件 ONBOOT=yes

# 查看修改的网络配置文件
cat /etc/sysconfig/network-scripts/ifcfg-eth1
```

###### hostnamectl 修改主机名 

---

`hostnamectl`用于显示`主机名`和一些系统相关的信息，主要用于`永久修改`主机名并且不需要重启系统。

```cython
# 命令语法
hostnamectl [选项][参数]

# 查看使用
hostnamectl
# -----------------------------------------------------
Static hostname: 主机名称

# 修改主机名称
hostnamectl set-hostname 新主机名
```



### 2、配置yum仓库

---

**试题概述：**

YUM 存储库已可以从 http://classroom.example.com/dvd/BaseOS 和 http://classroom.example.com/dvd/AppStream 使用配置您的系统，以将这些位置用作默认的存储库。

#### `方法1：手动配置`

考试时虚拟机`没有`安装`vim`编辑器，手动配置要使用`vi`编辑器（使用方法与vim 一样）

#### `方法2：命令配置`

repo文件是`yum`源（软件仓库）的`配置`文件，通常一个`repo文件`定义了一个或多个`软件仓库`的细节内容(记录了`包的下载路径`，告诉yum去哪里寻找将要下载的软件)

```cython
# 删除 yum配置文件夹 中的 配置文件(下面进行重新配置)
rm -rf /etc/yum.repos.d/*
```



`DNF`是一款Linux软件包管理器，用于管理`RPM`软件包。DNF的功能：1、可`查询`软件包信息  2、从`指定软件库`获取`软件包` 3、自动处理`依赖关系`

>DNF与YUM兼容，提供了YUM兼容的命令行以及扩展和插件提供的API。使用`DNF`需要`管理员权限`。

```cython
# 添加软件源    condif-manager:配置管理
dnf config-manager --add http://classroom.example.com/dvd/BaseOS
dnf config-manager --add http://classroom.example.com/dvd/AppStream
# -----------------------------------------------------
centos8发行版通过 BaseOS 和应用流 (AppStream) 仓库发布，BaseOS源是一个最小化系统所需要的包。AppStream 是对传统 rpm 格式的全新扩展，为一个组件同时提供多个主要版本。
```



`echo`会将`输入`的`字符串`送往标准输出。`>>`追加重定向，添加`gpgcheck=0`的配置到repo文件最后一行。

```cython
# 添加 gpgcheck=0 配置到BaseOS和AppStream文件内。关闭安全机制
echo gpgcheck=0 >> /etc/yum.repos.d/classroom.example.com_dvd_BaseOS.repo
echo gpgcheck=0 >> /etc/yum.repos.d/classroom.example.com_dvd_AppStream.repo
# 清除 YUM缓存
yum clean all
# 显示所有仓库
yum repolist 
# 配置好 yum源后，安装vim编辑器。（yum -y：对所有的提问都回答“yes”）
yum -y install vim

# -----------------------------------------------------
gpgcheck是gpg签名是用来在Linux实现官方发布的包的签名机制，主要为了软件下载的使用安全，1是开启，0是关闭，一般内部部署软件包可以关掉。
```



### 3、设置 SELinux

---

**试题概述：**

`非标准端口 82`上运行的`Web服务器`在提供内容时遇到问题。根据需要调试并解决问题，使其满足以下条件：

​	1、系统上的 `Web 服务器`能够提供 `/var/www/html` 中所有现有的 `HTML文件`（注：不要删除和改动现有内容）。

​	2、Web 服务器在`端口82`上提供内容。

​	3、Web 服务器在`系统启动`时`自动启动`。

---

SELinux极大的增强了Linux系统的安全性，能将`用户权限`关在笼子里，如`httpd服务`，Apache默认只能访问`/var/www`目录，并只能`监听`80和443端口，因此能有效防范0-day类的攻击。	

例子：系统上的`Apache`被发现存在一个`漏洞`，使得某远程用户可以访问系统上的敏感信息（比如/etc/passwd 来获取系统已存在用户），而修复该安全漏洞的Apache 更新补丁尚未推出。此时SELinux 可以起到`修补该漏洞`的`缓和`方案。因为`/etc/passwd`不具有Apache的访问标签。所以Apache对于`/etc/passwd`的访问会被SELinux阻止。

`80端口`用于www即万维网传输信息的协议。`81、82端口`即重定向端口、就是把一个端口重定向到另一个地址。实现重定向是为了隐藏公认的默认端口，减低受破坏率。

#### 实例

---

```cython
# 添加非标准化端口82
semanage port -a -t http_port_t-p tcp 82

# 修改安全上下文，并将文件的安全环境变换为指定环境
chcon -R --reference=/var/www/ /var/www/html/

# 查看文件的 安全上下文 的详细信息
ls -lZ /var/www/html/

# 重启网络服务
systemctl restart httpd

# 设置开机自启 网络服务
systemctl enable httpd
```

##### 解析：

---

###### semanage 查询与修改

---

semanage `查询`与`修改`SELinux默认目录的安全上下文。

常用参数：

- `port`：管理定义的网络端口类型。
- `-l`：列出所有记录
- `-a`：添加记录。
- `-t`：添加类型。
- `-p`：指定添加的端口是tcp或udp协议的，port子下使用。

```cython
# 列出http协议的所有端口
semanage port -l | grep http
# 添加非标准化端口82   http_port_t -p tcp 82为 添加端口82
semanage port -a -t http_port_t -p tcp 82

```

![image-20220928110801366](../../AppData/Roaming/Typora/typora-user-images/image-20220928110801366.png)

###### 安全上下文管理命令 chcon

---

`chcon`命令是`修改对象`（文件）的`安全上下文`，如：用户、角色、类型、安全级别。也就是==将`每个文件`的`安全环境`变更至`指定环境`。==

常用参数：

- `-R`：递归处理所有的文件及子目录	
- `--reference`选项时，把`指定文件`的`安全环境设置`为与`参考文件`相同。     

使用chcon命令，确保这些`新拷贝`的文件具有正确的`apache`应用的`SELinux安全上下文`。

```cython
# 参考语法（非官方）：
chcon -R --reference=参考文件 指定文件

# 输出结果(仅变化部分)--------------------------------------------------------
unconfined_u : object_r : user_home_t
system_u : object_r : public_content_t
```

`输出结果：`

>​	信息被三个`冒号`分开，分别为是`身份识别`：`角色`：`类型`。身份识别中常见的情况主要分别:
>
>`unconfined_u(自由的)` 指的是`不受限`的用户，就是说该文件来自于`不受限的进程所产生`的。
>
>`system_u(系统的)` 指`系统自己产生`的文件，简单说是`网络服务`或`系统运行所产生的文件`会被``识别``为`system.u`。
>
>



### 4、配置用户账户

---

**试题解析：**

创建下列用户、组、组成员资格：

​	1、名为`sysmgrs`的组。

​	2、用户 `natasha`，作为`次要组`从属于`sysmgrs`。

​	3、用户 `harry`，作为`次要组`从属于`sysmgrs`。

​	4、用户 `sarah`，`无权访问`系统上的交互式`shell`且不是`sysmgrs组`的成员。

​	5、密码都为`123`。

#### 实例：

```cython
# 添加用户组（/ect/group 文件下查看）
groupadd sysmgrs
# 创建用户账号，并指定用户所属附加群组 | 指定用户的附加组 -G 附加组 用户
useradd -G sysmgrs natasha
useradd -G sysmgrs harry
# 创建用户账号，并指定用户登陆后使用的shell | -s代表用户指定的shell，一般为/usr/bin
useradd -s /bin/false sarah   # bin 存放的是系统命令
# 设置用户密码(使用shell脚本循环)   格式 echo “新密码”|passwd --stdin 用户名
for i in natasha harry sarah; do echo 123 | passwd --stdin $i;done
```

### 5、配置 cron计划任务

---

**试题概述：**

​	配置cron作业，该作业在本地时间`每天14.23`运行并执行以下命令：

​	`logger “EX200 in progress”`,以用户 `natasha`身份运行

---

```cython
# 设置用户natasha 每天14：23 运行 作业程序
# -e 表示 编辑crontab   | -u 表示 编辑/查看/删除某用户的crontab
crontab -e -u natasha

# 文本添加
> 23 14 crontav logger "EX200 in progress"

```

![image-20221003161031938](https://www.hualigs.cn/image/633a9913eba0f.jpg)

>几个特殊情况和符号：
>
>- `/` 代表`时间间隔`，比如在`分钟`里面，`0-30/2` 表示`前0-30分钟之内 每2分钟`执行一次。
>
>  也可以和 `*` 结合，`* / 2` 表示`每两分钟执行一次`
>
>- `-` 连字符表示`值的范围`，在`分钟`里面1-30 代表 1到30分钟



### 6、创建目录

---

**试题概述：**

​	1、创建`目录` /home/managers

​	2、/home/managers 的`组用权`是sysmgrs

​	3、`目录`应当可被`sysmgrs`的成员读取，写入和访问，但任何其他用户不具有这些权限。

​	4、/home/managers 中`创建的文件`自动将组所有权设置到`sysmgrs`组

```cython
# 创建 目录
mkdir /home/managers
# chown 用于设置 文件所有者 和 文件关联组的命令 | 语法格式：chown user[:group] 目录或文件
# 将 /home/managers 的关联组设置为 sysmgrs
chown :sysmgrs /home/managers
# g表示 所属组 对 文件所拥有的权限。r：可读 w：可写 x：可执行
# o表示 其他人对该文件所拥有权
# /home/managers可被所属组的用户进行全部操作，其他人（非所属组的用户）无权限
# chmod 控制用户对文件的权限
chmod g=rwx,o=- /home/managers
# 给 /home/managers 设置组标识符，将新加入的组用权都加入sysmgrs。
chmod g+s /home/managers/
```



### 7、配置 NTP 时间同步

---

**试题概述：**

​	配置您的系统，使其称为 `classroom.example.com` 的NTP客户端。

---

NTP：用来使计算机时间同步的一种协议

`chrony` 是一个开源软件，是一个用来`维持计算机系统时钟准确性`的程序。默认配置文件在`/etc/chrony.conf`。它能保持系统时间与时间服务器(NTP)同步，让时间始终保持同步

`chronyd` 是一个系统后台运行的守护进程，他根据网络上其他时间服务器时间来测量本机时间的偏移量从而调整系统时钟。

`chronyc`：提供一个用户界面，用于监控性能并进行多样化的配置。用来监控chronyd性能和配置其参数的用户界面，也可以在一台不同的远程计算机上工作。

```cython
# 本题有陷阱，红帽故意把/etc/chrony.conf的改错了。删掉并重新装
rm -rf /etc/chrony.conf
yum - y reintsall chrony
# 编辑文件
vim /etc/chrony.conf
# 修改 pool(集中) iburst（爆发）
> pool classroom.example.com iburst
# 重启chronyd服务
systemctl restart chronyd
# 设置开机自启动
systemctl enable chronyd
# 查看时间同步源
chronyc sources -v

```

### 8、配置 autofs

---

**试题概述：**

配置 `autofs`，按照如下所述`自动挂载远程用户的主目录`：

​	1、 classroom.example.com(172.25.0.254) NFS 导出 /rhel8 到您的系统。此文件系统包含为用户 user1 预配置的主目录。

​	2、 user1 的主目录是 classroom.example.com:/rhel8/user1。

​	3、 user1 的主目录应自动挂载到本地 /home/rhel8 下的 /home/rhel8/user1

​	4、 主目录必须可供其用户写入

​	5、 user1 的密码是 123

---

autofs（自动挂载）是一种系统守护进程，我们可以把挂载信息写入其配置文件中，如果用户不访问其他存储介质的，则系统不会进行挂载，如果用户尝试访问该存储介质，则autofs会自动进行挂载操作，上述所有操作对用户而言是透明的，这样一来，autofs服务节省了服务器的网络和硬件资源。

```cython
# 安装 autofs
yum -y install autofs
# 本地目录：/home/rhe18    规则文件：/etc/auto.rule
# auto 自动挂载配置文件：auto.master
vim /etc/auto.master
# 本地目录 规则文件
> /home/rhel8 /etc/auto.rule
# 用户 权限 nfs路径
# 用户：user1  权限：-rw  nfs路径：classroom.example.com:/rhel8/user1
vim /etc/auto.rule
> user1 -rw classroom.example.com :/rhel8/usr1
# 重启启动 autofs服务
systemctl restart autofs
# 自启动 autofs服务
systemctk enable autofs

```



### 9、设置文件权限

---

**试题概述：**

将文件 `/etc/fstab` 复制到 `var/tmp/fstab`。配置 `var/tmp/fstab` 的权限满足以下条件：

​	1、文件 `var/tmp/fstab` 自 root 用户所有

​	2、文件 `/var/tmp/fstab` 属于组 `root`

​	3、文件 `/var/tmp/fstab` 应不能被任何人执行

​	4、用户 `natasha` 能够读取和写入 `/var/tmp/fstab`

​	5、用户 `harry` 无法读取和写入 `/var/tmp/fstab`

​	6、所有其他用户（当前或未来）能够读取 /var/tmp/fstab

---

```cython
# 将文件 /etc/fstab 复制到 /var/tmp/fstab
cp /etc/fstab /var/tmp/fstab
# setfacl 用来细分 Linux的文件权限
# hmod命令可以把文件权限分为u,g,o三个组，而setfacl可以对每一个文件或目录设置更精确的文件权限
# -m：设置后续acl参数
setfacl -m u:natasha:rw /var/tmp/fstab
# 用户 harry 无法读写
setfacl -m u:harry:- /var/tmp/fstab
# 
```

### 10、创建用户账户

---

**试题概述：**

创建用户 `user2`，其他`用户 ID`为`3388`

用户`密码`为123

```cython
# 添加系统新用户 	语法：useradd [选项] 用户名
# -u:指定用户的UID（不小于500）
useradd -u 3388 user2
echo 123 | passwd --stdin user2

# 查看所有用户
 /etc/gshadow
```

### 11、查找文件

---

**试题概述：**

​	`查找 student` 所有的文件并将其`复制`到`/root/dfiles` 目录下.

```cython
# 陷阱：1、/root/dfiles 没有文件  2、cp 要加参数 -a
mkdir /root/dfiles
# 将 student 所有文件复制到 /root/dfiles

```

```
==================================解析==================================
# -user 按所有者查找。所有者为student
find /-user student 
# 按文档类型查找（f：文件，d：目录 l：快捷方式）|这里是查找文件
-type f
# -exec：将前一个命令 输出作为参数 传给第二个命令
# cp -a为文件属性不发生改变
# {}：为源文件,前面find搜索的文件的全部文件
# 目的文件：/root/dfiles
# \：结束

```

### 12、查找字符串

---

**试题概述:**

查找 `xxx` 目录下包含 `yyy` 字符串的文件，将所有这些`行`的`副本`按`原始顺序`放在目录`/root/files`中。

```cython
# 创建目录
mkdir /root/files
# 将 xxx目录下 yyy字符串的文件，放到目录/root/files中
find xxx -name *yyy* -type f -exec cp -p {} /root/files \;
```

### 13、创建存档

---

**试题概述：**

创建一个名为 `/root/books.tar.gz`的tar存档，其应包含`/usr/local`的`tar`存档，其应包含 `/usr/local` 的内容。该 tar存档必须使用`bzip2`进行压缩

```
# -j:有bz2属性的 | -c:创建压缩存档（档案）| -f 使用存档名字（档案名字）。这个参数是最后一个，后面接存档名
# -P:保留备份数据的原本权限与属性
tar -jPcf /root/book.tar.gz /usr/local
```



### 





