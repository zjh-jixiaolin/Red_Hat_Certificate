# RHCE — 红帽认证工程师

---

[TOC]

## 0. 重要配置信息

### 系统信息

---

考试期间，除了您就坐位置台式机之外，还将使用多个虚拟系统。您不具有台式机系统的 root 访问权，但具有对虚拟系统的完整 root 访问权

考试中，所有的操作都在 `control（控制节点）` 上进行操作。

|         系统          | IP地址           | Ansible 角色                       |
| :-------------------: | ---------------- | ---------------------------------- |
| `control（控制节点）` | `172.25.250.254` | `ansible control node（控制节点）` |
|  `node1（受控节点）`  | `172.25.250.9`   | `ansible managed node（受控节点）` |
|  `node2（受控节点）`  | `172.25.250.10`  | `ansible managed node（受控节点）` |
|  `node3（受控节点）`  | `172.25.250.11`  | `ansible managed node（受控节点）` |
|  `node4（受控节点）`  | `172.25.250.12`  | `ansible managed node（受控节点`   |
|  `node5（受控节点）`  | `172.25.250.13`  | `ansible managed node（受控节点）` |

这些系统的 IP 地址采用静态设置。请勿更改这些设置。

主机名称解析已配置为解析上方列出的完全限定主机名，同时也解析主机短名称（ping通主机短名称和IP地址）。

>① control 为控制节点，node1 - node5 为受控节点
>
>② 所有操作都在 control 控制节点上进行操作，不要在受控节点操作
>
>③ 考试时，IP地址可能为主机名或其他，随机应变

### 账户信息

所有系统的 `root` 密码是 `flectrag`，请勿修改。

为方便起见，所有系统上预装了 `SSH密钥`，允许在`不输入密码`的前提下通过 SSH进行 `root访问`，请不要对系统上的root SSH 配置文件进行修改。

>SSH密钥 是一种 无须密码登录Linux实例的认证方式。

`Ansible 控制节点` 上已`创建`了 用户`账户 greg`。此账户`预装`了` SSH密钥`，允许在 Ansible 控制节点和各个 Ansible 受管节点 之间进行SSH登录。请勿对系统上的 greg SSH 配置文件进行修改。你可以从 root 账户使用 su 访问此用户账户。

>**重要信息**
>
>除非另有所指，否则所有工作（包括 `Ansible playbook、配置文件和主机清单`等）应当保持在控制节点上的目录 `/home/greg/ansible` 中，并且应当归 `greg用户`所有。所有 Ansible 相关的命令应当由 greg用户从 Ansible 控制节点上的这个目录运行。

```shell
# ssh密钥
[root@control ~]# ssh root@172.25.250.254
# Ansible控制节点已创建了用户greg。此账户预装了SSH密钥
[greg@control ~]$ ssh greg
---------------------------------------------------------
# 第二种登录方式
[kiosk@foundation0 ~]$ ssh greg@172.25.250.254

> 所有的操作（包括Ansible playbook、配置文件和主机清单）都是以控制节点上的greg用户去完成。
```

### 其他信息

一些考试项目可能需要修改 Ansible 主机清单。您要负责确保所有以前的清单组和项目保留下来，与任何其他更改共存。您还要有确保清单中所有默认的组和主机保留您进行的任何更改。

<font color='red' size=4 >   考试系统上的防火墙默认为不启用，SELinux 则处于强制模式。 </font>

产品文档位置：

- http://materials/docs

>**重要信息**
>
>评分前，您的`Ansible收管节点（node1 - node5）系统`将重置为考试开始时的初始状态，您的 Ansible playbook 将通过以 `greg`用户省份从控制节点上的目录`/home/greg/ansible` 目录来运行应用。在 playbook 运行后，系统会对您的受管节点进行评估，以判断它们是否按照规定进行了配置。（评分前会重置所有的受控节点，所以要在控制节点上进行操作）

### 考试注意点

- `考试时间：4小时`
- `考试时，root密码会发生改变`
- `题目：15道题`
- `评分之前，会先还原所有托管节点到初始状态，然后执行/home/用户/ansible中的playbook（剧本）和脚本，然后再监测托管节点上是否达到题目要求。`
- `评分脚本——验证（考试用不了，练习时使用）：exam-grade。`

- `满分300分，合格：210分，附加题也包含在300分。`

<font color='red' size=4 >   RHCE全程是控制节点操作其他节点，使用的账号是greg，所有系统的root密码是 flectrag </font>





<br />

## 1. 安装和配置 Ansible

按照下述所述，在控制节点 `control`上安装和配置 `Ansible`：

- `安装`所需的软件包 

- 创建名为 `/home/greg/ansible/inventory`的`静态清单文件`，以满足以下要求：

  - `node1` 是 `dev` 主机组成员
  - `node2` 是 `test` 主机组的成员
  - `node3` 和 `node4` 是 `prod`主机组的成员
  - `node5` 是 `balancers` 主机组的成员
  - `prod` 组是 `webservers` 主机组的成员

  

- 创建名为`/home/greg/ansible/ansible.cfg`的配置文件，以满足以下要求：
  - 主机清单文件为`/home/greg/ansible/inventory`
  - playbook中使用的角色的位置包括 `/home/greg/ansible/roles`

### Course

```shell
# 远程连接 greg用户 进入控制节点control
ssh greg@control
> 输入密码

# 安装 ansible 软件包
sudo yum -y install ansible

# 创建角色路径，并进入 ansible 目录   (mkdir -p：递归创建目录)
mkdir -p /home/greg/ansible/roles

# 据题编辑清单文件 inventory
cd ansible
vim /home/greg/ansible/inventory
------------------------------------
[dev]
node1

[test]
node2

[prod]
node3
node4

[balancers]
node5

[webservers:children]  # 嵌套组
prod
------------------------------------
# ① 验证 --graph：以图标形式显示
ansible-inventory -i inventory --graph

# 安装完ansible会有一个默认的配置文件，正好满足题目要求。复制ansible.cfg 配置文件    
# cp -r：递归（recurse）复制	cp -f：强制复制
cp -rf /etc/ansible/ansible.cfg .
 # 修改当前用户配置文件 ansible.cfg
vim ansible.cfg
------------------------------------
# 默认配置
[defaults]
# 主机清单文件为 /home/greg/ansible/inventory
inventory = /home/greg/ansible/inventory
# playbook 中使用角色位置包括 /home/greg/ansible/roles
roles_path = /home/greg/ansible/roles
# 关掉key检查
host_key_checking = False
# 更改远端执行用户为题目指定的用户，这里是greg，考试时会变化
remote_user = greg

# 权限升级
# 任务为自动使用sudo命令从 greg 切换到 root
[privilege_escalation]
# 开启切换用户
become = true
# 如何切换用户（方法）：sudo
become_method = sudo 
# 要在受控主机（node1-node5）上切换到的用户（切换受控节点时是使用root用户）
become_user = root
# 使用become_method切换用户时是否需输入密码（true 和 false）
become_ask_pass = false
------------------------------------
# ② 验证
ansible all -m ping -o 

```

### Verify

![image-20230323192006257](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303231920942.png)

![image-20230323192233941](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303231922324.png)



<br />



## 2.  创建和运行 Ansible 临时命令

---

作为系统管理员，您需要在受管节点上安装软件。

请安按照正文所述，创建一个名为`/home/greg/ansible/adhoc.sh`的shell 脚本，该脚本将使用Ansible 临时命令在各个受管节点上安装yum 存储库：

存储库1：

- 存储库的名称为 `EX294_BASE`
- 描述为`EX294 base software`
- 基础URL为`http://content/rhel8.0/x86_64/dvd/BaseOS`
- GPG 签名检查为 `启动状态`
- GPG 密钥 URL为 `http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release`
- 存储库为`启动状态`

存储库2：

- 存储库的名称为 `EX294_STREAM`
- 描述为 `EX294 stream software`
- 基础 URL为 `http://content/rhel8.0/x86_64/dvd/AppStream`
- GPG签名检查为`启动状态`
- GPG 密钥 URL 为 `http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release`
- 存储库为`启动状态`

### Course

```shell
# 此题为在收管节点（node1 - node5）上安装软件 —— 在受控节点（node1-node5）上配置yum仓库

# ansible-doc：查询文档
ansible-doc -l | grep yum
> yum
> yum_repository

# 帮助案例
ansible-doc yum_repository

# 编写yum存储库脚本
vim /home/greg/ansible/adhoc.sh
------------------------------------
# -m：指定使用模块    -a：指定参数
#!/bin/bash    
ansible all -m yum_repository -a "name='EX294_BASE' \
        description='EX294 base software' \
        baseurl='http://content/rhel8.0/x86_64/dvd/BaseOS' \
        gpgcheck=yes \
        gpgkey='http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release' \
        enabled=yes"

ansible all -m yum_repository -a "name='EX294_STREAM' \
        description='EX294 stream software' \
        baseurl='http://content/rhel8.0/x86_64/dvd/AppStream' \
        gpgcheck=yes \
        gpgkey='http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release' \
        enabled=yes"
------------------------------------
# 给脚本添加执行权限（查看权限使用ll，权限原本为-rw-rw-r--，执行下方命令后-rwxrwxxr-x）
# 给shell脚本添加权限，并运行
chmod +x adhoc.sh
# 运行两次，第一次黄色代表改变，第二次绿色则代表没有出错
./adhoc.sh

# 验证
ansible all -a "yum repolist"
# 第二个验证方法（不确定时再用）
ansible all -a "yum -y install lftp"
```

### Verify

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303231946097.png" alt="image-20230323194634588" style="zoom: 80%;" />

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303231950623.png" alt="image-20230323195017973" style="zoom:67%;" />



<br />



## 3. 安装软件包

**安装软件包：**

创建一个名为 `/home/greg/ansible/packages.yml` 的 playbook（剧本文件）：

- 将 `php`和`mariadb`软件包安装到`dev、test`和`prod`主机组中的主机上
- 将`RPM Development Tools`软件包组安装到`dev`主机组的主机上
- 将`dev`主机组中主机上的`所有软件包更新为最新版本`

### Course

```shell
# 这题是关于剧本文件，playbook是ansible用于配置，部署和管理被控节点的剧本（这题的软件和软件包组要注意区分）
# 帮助文档
ansible-doc yum
# 编写剧本
vim /home/greg/ansible/packages.yml
------------------------------------
# 使用yum源安装软件时，提供两种安装方式：present和latest
使用 present 只会确保安装软件，不会关注版本，只要软件不存在则会安装
使用 latest 当软件源中的安装包有版本更新时，latest会走动将软件升级为最新版本。

# 1、安装php,mariadb软件包到指定主机上
---
- name: p1
  hosts: dev,test,prod
  tasks:
    - name: install php and mariadb
      yum:
        name: php,mariadb
        state: present

# 2、dev主机组安装软件包组 和 更新软件包为最新
- name: p2
  hosts: dev
  tasks:
    - name: install Development Tools Groups
      yum:
        name: "@RPM Development Tools"
        state: present

    - name: update all
      yum:
        name: "*"
        state: latest
------------------------------------
# 验证 (ansible-playbook：定制自动化任务)
# 运行脚本（运行两次：全绿即可，出现红的可能错误）
ansible-playbook package.yml

# 更完整的验证方式 
ansible dev,test,prod -a "yum info php"
ansible dev,test,prod -a "yum info mariadb"
ansible dev -a "yum grouplist"  # 验证软件包组
ansible dev -a "yum update"

```

### Verify

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303232006168.png" alt="image-20230323200553034" style="zoom:80%;" />

![image-20230323201150567](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303232011240.png)

更完整的验证方式：

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303232014047.png" alt="image-20230323201404656" style="zoom: 67%;" />

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303232015428.png" alt="image-20230323201519369" style="zoom: 67%;" />

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303232016288.png" alt="image-20230323201628819" style="zoom: 80%;" />

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303232017990.png" alt="image-20230323201726350" style="zoom:80%;" />



<br />



## 4. 使用 RHEL 系统角色

**使用 RHEL 系统角色**

安装 RHEL 系统角色软件包， 并创建符合以下条件的 playbook `/home/greg/ansible/timesync.yml`：

- 在`所有受管节点`上运行
- 使用 `timesync` 角色
- 配置该角色，以使用当前有效的 `NTP` 提供商
- 配置该角色，以使用时间服务器 `172.25.254.254`
- 配置该角色，以启用 `iburst` 参数

### Course

```shell
# 搜索软件包（yum search：查找软件包命令）
sudo yum search roles
> rhel-system-roles.noarch    # 搜索结果有一个关于系统角色的软件包

# 安装系统角色
sudo yum -y install rhel-system-roles.noarch

# 查看系统角色的安装路径和产生的文件
# rpm -q：查询软件包是否安装  rpm -l:查询软件包列表
rpm -ql rhel-system-roles-1.0-5.el8.noarch 
> /usr/share/ansible/roles   # 安装的系统角色路径
> /usr/share/doc/rhel-system-roles/timesync/example-timesync-playbook.yml # timesync案例

# 在配置文件anisble.cfg 中关联刚刚安装的系统角色路径
vim ansible.cfg
------------------------------------
> roles_path = roles_path = /home/greg/ansible/roles:/usr/share/ansible/roles 
# 验证（查看系统角色列表）
ansible-galaxy list
------------------------------------

# 查询系统时间角色的案例
rpm -ql rhel-system-roles-1.0-5.el8.noarch | grep example
# 复制案例，修改名字
cp -rf /usr/share/doc/rhel-system-roles/timesync/example-timesync-playbook.yml ./timesync.yml
# 据题编写案例
------------------------------------
---
# 在所有收管节点上运行
- hosts: all
   vars:
# 配置该角色，使用当前有效的NTP提供商
# 配置该角色，以使用时间服务器 172.25.254.254
# 配置该角色， 以启用 iburst 参数
   timesync_ntp_servers:
      - hostname: 172.25.254.254
        iburst: yes
   roles:
# 使用 timesync 角色
      - rhel-system-roles.timesync
------------------------------------
# 运行playbook
ansible-playbook timesync.yml
# 验证
ansible all -a 'timedatectl'
```

### Verify

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303232050450.png" alt="image-20230323204931487" style="zoom:80%;" />



<br />

## 4.1 扩展题

**使用 RHEL 系统角色**

安装 RHEL 系统角色软件包，并创建符合以下条件的 playbook ` /home/greg/ansible/selinux.yml`：

- 在 `所有受管节点`上运行
- 使用`selinux` 角色
- 配置该角色，配置被管理节点的selinux 为 `enforcing(强制执行)`

### Course

```shell
# 搜索软件包
yum search roles

# 安装系统角色软件包
sudo yum -y install rhel-system-roles.noarch

# 查看安装系统角色路径，添加到配置文件
rpm -ql rhel-system-roles-1.0-5.el8.noarch 
> /usr/share/ansible/roles   # 安装的系统角色路径

# 在配置文件anisble.cfg 中关联刚刚安装的系统角色路径
vim ansible.cfg
------------------------------------
> roles_path = roles_path = /home/greg/ansible/roles:/usr/share/ansible/roles 
# 验证（查看系统角色列表）
ansible-galaxy list
------------------------------------

# 查询系统selinux案例
rpm -ql rhel-system-roles-1.0-5.el8.noarch | grep example
> /usr/share/doc/rhel-system-roles/selinux/example-selinux-playbook.yml

# 复制案例到指定路径，并据题编写剧本
------------------------------------
---
- hosts: all # 在所有收管节点
  vars:
    selinux_state: enforcing # 强制模式
  roles: 
    - rhel-system-roles.selinux # 使用selinux角色
------------------------------------
# 执行playbook
ansible-playbook selinux.yml

# 验证
ansible all -a "grep ^SELINUX /etc/selinux/config" 

```



### Verify

![image-20230324093651891](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303240936295.png)



<br />



## 5. 使用 Ansible Galaxy 安装角色

**使用 Ansible Galaxy 安装角色**

使用 Ansible Galaxy 和要求文件 `/home/greg/ansible/roles/requirements.yml`。从以下 URL 下载角色 并安装到 `/home/greg/ansible/roles`：

- ` http://materials/haproxy.tar`，此角色的名称为 `balancer`
- `http://materials/phpinfo.tar`，此角色的名称为 `phpinfo`

### Course

`Ansible Galaxy` 指的是一个网站共享和下载 `Ansible` 角色，也可以是帮助 `roles（角色）`更好的工作的命令行工具

```shell
# 根据题编写剧本
vim /home/greg/ansible/roles/requirements.yml
------------------------------------
# src：源文件位置
---
- src: http://materials/haproxy.tar
  name: balancer
  
- src: http://materials/phpinfo.tar
  name: phpinfo
------------------------------------

# 使用 Ansible Galaxy 安装角色
# install -r：强制安装   -p：指定安装的路径
ansible-galaxy install -r /home/greg/ansible/roles/requirements.yml -p /home/greg/ansible/roles

# 检查：可省略（进入/ansible/roles目录下查看）
ls
pwd   # 检查是否安装到题目指定路径

# 验证
ansible-galaxy list # 查看角色列表
```

### Verify

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303241018238.png" alt="image-20230324101826060" style="zoom:80%;" />

![image-20230324101915151](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303241019768.png)



<br />



## 6.  创建和使用角色

**创建和使用角色（两种情况，见下方框）**

根据下列要求，在 `/home/greg/ansible/roles` 中创建名为 `apache` 的角色：

- httpd 软件包已安装，设为 `系统启动时启用` 并 `启动`‘
- `防火墙`已启用并正在运行，并使用允许访问 `Web` 服务器的规则
- 模板文件 `index.html.j2` 已存在，用于创建具有以下输出的文件 `/var/www/html/index.html `：

```
Welcome to HOSTNAME on IPADDRESS
```

其中，`HOSTNAME` 是受管节点的`完全限定域名`，`IPADDRESS` 则是受管节点的 `IP 地址`。

创建一个名为 `/home/greg/ansible/apache.yml` 的 playbook：

- 该 play 在 `webservers` 主机组中的主机上运行并将使用 apache 角色

### Course

```shell
# 进入角色路径，创建名为 apache 的角色
cd /home/greg/ansible/roles
ansible-galaxy init apache

# 编写任务 tasks 文件
vim apache/tasks/main.yml 
------------------------------------
# 题目提示：httpd软件包已安装（这里我们需要编写一个任务来确保软件包是否安装）
# 帮助文档：ansible-doc yum
---
- name: install apache
  yum:
    name: httpd
    state: latest

# 启动 Apache服务
# 帮助文档： ansible-doc service
- name: Start service httpd
  service:
    name: httpd
    state: started  # 保持服务状态为（启动）
    enabled：yes # 将该服务添加到自动启动的服务列表中

# 题目提示：防火墙已启用并正在运行（这里我们需要编写任务来确保防火墙启用并运行）
- name: Start service firewalld
  service:
    name: firewalld
    state: started  # 保持服务状态为（启动）
    enabled：yes # 将该服务添加到自动启动的服务列表中

# 打开防火墙80端口，以运行传入 http 流量（使用允许访问 Web 服务器的规则）
# 帮助文档： ansible-doc firewalld
- name: open firewalld port 
  firewalld:
    service: http
    permanent: yes  # 设置80端口永久打开（规则持久化，重启后仍然生效）
    immediate: yes # 立即生效（指定规则立即生效，不需要等待防火墙服务重启）
    state: enabled # 启用

# 该任务使用template模块，该模块将从index.html.j2模板文件中渲染内容，并将结果复制到/var/www/html/index.html文件中
# 帮助文档：ansible-doc template
- name: Template a file 
  template:
    src: index.html.j2  # 模板的源文件路径
    dest: /var/www/html/index.html  # 复制生成文件的目标位置
------------------------------------
__________________________________________________________________________________________________

# 编写模板文件
# 完全限定域名：ansible node2 -m setup -a "filter=*fqdn*"
# IPADDRESS: ansible node2 -m setup -a "filter=*ipv4*"
vim apache/templates/index.html.j2
------------------------------------
Welcome to {{ ansible_fqdn }} on {{ ansible_default_ipv4.address }}
------------------------------------

__________________________________________________________________________________________________

# 编写playbook文件进行验证
vim /home/greg/ansible/apache.yml
------------------------------------
---
- name: p1
  hosts: webservers
  roles:
    - apache
------------------------------------

# 执行 + 验证
ansible-playbook /home/greg/ansible/apache.yml
curl node3
> Welcome to node3.lab.example.com on 172.25.250.11
curl node4
>  Welcome to node4.lab.example.com on 172.25.250.12
```

### Verify

![image-20230325175216587](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303251752098.png)



<br />



## 7. 从 Ansible Galaxy 使用角色

**从 Ansible Galaxy 使用角色**

根据下列要求，创建一个名为 `/home/greg/ansible/roles.yml` 的 playbook ：

- playbook 中包含一个 play， 该 play 在 `balancers` 主机组中的主机上运行并将使用 `balancer` 角色。

  - 此角色配置一项服务，以在 `webservers` 主机组中的主机之间平衡 Web 服务器请求的负载。

  - 浏览到 balancers 主机组中的主机（例如 http://172.25.250.13 ）将生成以下输出：

    ```
    Welcome to serverb.lab.example.com on 172.25.250.11
    ```

    

  - 重新加载浏览器将从另一 Web 服务器生成输出：

    ```
    Welcome to serverb.lab.example.com on 172.25.250.12
    ```

- playbook 中包含一个 play， 该 play 在 `webservers` 主机组中的主机上运行并将使用 `phpinfo` 角色。

  - 请通过 URL `/hello.php` 浏览到 `webservers` 主机组中的主机将生成以下输出：

     ```shell
     Hello PHP World from FQDN
     ```

  - 其中，FQDN 是主机的完全限定名称。

    ```shell
    Hello PHP World from serverb.lab.example.com
    ```

    另外还有 PHP 配置的各种详细信息，如安装的 PHP 版本等。

  - 同样，浏览到 `http://172.25.250.12/hello.php`会生成以下输出：

    ```
    Hello PHP World from serverc.lab.example.com
    ```

    另外还有 PHP 配置的各种详细信息，如安装的 PHP 版本等。
    
    >（题型2：第二种情况）
    >
    >从 Ansible Galaxy 使用角色
    >
    >根据下列要求，创建一个名为 /home/greg/ansible/roles.yml 的 playbook ：
    >
    >playbook 中包含一个 play， 该 play 在 balancers 主机组中的主机上运行并将使用 balancer 角色。
    >
    >此角色配置一项服务，以在 webservers 主机组中的主机之间平衡 Web 服务器请求的负载。
    >
    >浏览到 balancers 主机组中的主机（例如 http://172.25.250.13 ）将生成以下输出：
    >
    >Welcome to node3.lab.example.com on 172.25.250.11
    >
    >重新加载浏览器将从另一 Web 服务器生成输出：
    >
    >Welcome to node4.lab.example.com on 172.25.250.12
    >
    >playbook 中包含一个 play， 该 play 在 webservers 主机组中的主机上运行并将使用 phpinfo 角色。
    >
    >请通过 URL /hello.php 浏览到 webservers 主机组中的主机将生成以下输出：
    >
    >Hello PHP World from FQDN
    >
    >其中，FQDN 是主机的完全限定名称。
    >
    >Hello PHP World from node3.lab.example.com
    >
    >另外还有 PHP 配置的各种详细信息，如安装的 PHP 版本等。
    >
    >同样，浏览到 http://172.25.250.12/hello.php 会生成以下输出：
    >Hello PHP World from node4.lab.example.com
    >
    >另外还有 PHP 配置的各种详细信息，如安装的 PHP 版本等。

### Course

```shell
 # 据题编写剧本
 vim /home/greg/ansible/roles.yml
 ------------------------------------
# 此角色配置一项服务，以在 `webservers 主机组`中的主机之间`平衡 Web服务器请求的负载`
- name: a2
  hosts: webservers
  roles:
    - apache

 # playbook 中包含一个 `play`，该 play 在 `balancers 主机组`中的`主机上`运行并将使用 `balancer` 角色。
- name: a1
  hosts: balancers
  roles:
    - balancer

# playbook 中包含一个 play，该 play在 `webservers 主机组`中的主机上运行并将使用phpinfo角色。
- name: a3
  hosts: webservers
  roles:
    - phpinfo
------------------------------------    
  # 验证
  ansible balancers -a "curl node5"
```

### Verify

![image-20230326171438976](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303261714380.png)



<br />



## 8. 创建和使用逻辑卷

**创建和使用逻辑卷**

创建一个名为 `/home/greg/ansible/lv.yml` 的playbook，它将在`所有受管节点` 上执行下列任务：

- 创建符合以下要求的逻辑卷：

  - 逻辑卷创建在 `research` 卷组中
  - 逻辑卷名称为 `data`
  - 逻辑卷大小为 `1500 MiB`

- 使用 `ext4` 文件系统格式化逻辑卷

- 如果无法创建请求的逻辑卷大小，应显示错误信息

  ```shell
  Could not create logical volume of that size
  ```

  ，并应改为使用大小 `800 MiB`。

- 如果卷组 `research` 不存在，应显示错误信息

  ```shell
  Volume group done not exist
  ```

- 不要以任何方式挂载逻辑卷

### Course

```shell
# 创建逻辑卷的过程
物理卷 —— 卷组vg —— 逻辑卷lv —— 格式化逻辑卷 —— 挂载逻辑卷


# 创建逻辑卷的帮助文档
ansible-doc -l | grep LVM -----> ansible-doc lvol（查询创建逻辑卷的文档）
# 格式化帮助文档
ansible-doc filesystem
# 调试模块帮助文档
ansible-doc debug         
# 设置条件帮助文档
ansible-doc stat


# 据题创建playbook，并在所有受管节点上运行
vim /home/greg/ansible/lv.yml
 ------------------------------------
---
- name: create lvm
  hosts: all  # 在所有受管节点上运行
  tasks:
    # 任务块，会按照顺序执行，除非发生错误
    - block:  
       - name: Create volume1
         lvol: # 用于创建逻辑卷的 Ansible 模块
           vg: research  # 逻辑卷的卷组名称
           lv: data  # 逻辑卷名称
           size: 1500 # 逻辑卷大小（以MB为单位，如果考试为1GB，则写为1024MB）
       
       # 格式化逻辑卷
       - name: Create filesystem1
         filesystem:
             fstype: ext4 # 文件系统格式化类型
             dev: /dev/research/data  # 需要文件系统格式化的设备名
	
	# 如果block任务块出现错误，执行rescue异常模块下方任务
      rescue:
         - debug: # 打印一条消息到标准输出，表示无法创建指定大小的逻辑卷
             msg: Could not create logical volume of that size  
           when: ansible_lvm.vgs.research is defined # 当逻辑卷 存在时 执行
           
         - name: Create volume2
           lvol:
             vg: research
             lv: data
             size: 800        
           when: ansible_lvm.vgs.research is defined # 当逻辑卷 存在时 执行
           
         - name: Create filesystem2
           filesystem:
               fstype: ext4
               dev: /dev/research/data
           when: ansible_lvm.vgs.research is defined
           ignore_errors: yes # 执行任务遇到错误时不会终止playbook的执行(确保后方的debug能执行)

         - debug:
             msg: Volume group done not exist
           when: ansible_lvm.vgs.research is not defined # 指定逻辑卷 不存在 时执行
 ------------------------------------
 # 执行脚本并验证
 ansible-playbook lv.yml  # 下方红框区域不报错，并可看到changed则为成功
 # 验证
ansible all -a "vgs"  # 查看卷组
 ansible all -a "lvs"  # 查看逻辑卷
 ansible all -a "blkid /dev/research/data" # 查看文件系统类型
```



<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303151850849.png" alt="image-20230315185032015" style="zoom:80%;" />

### Verify

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303262005558.png" alt="image-20230326200501363" style="zoom:150%;" />

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303151931242.png" alt="image-20230315193117506" style="zoom:80%;" />

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303262002024.png" alt="image-20230326200249819" style="zoom: 80%;" />

![image-20230326200611171](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303262006068.png)

![image-20230326200811044](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303262008757.png)



<br />



## 8.1 创建和使用分区

**题型1**

**创建和使用分区（创建并使用磁盘分区**）

创建一个名为 `/home/greg/ansible/partition.yml` 的playbook，它将在 `所有受管节点`上创建分区：

- 在`vdb` 创建一个`1500MiB`的主分区，分区号为`1`，
- 使用`ext4`文件系统格式化，
- 将文件系统挂载到 `/newpart`



- 在 `vdc` 创建一个`1500MiB`的主分区，分区号为`2`，

- 使用 `ext4` 文件系统格式化

- 将文件系统挂载到 `/newpart1`

- 如果分区大小不满足，报错信息如下：

  ```shell
  Could not create partaion of that size
  ```

  ，改为创建一个`800 MiB` 的分区

- 如果磁盘不存在，报错信息如下：

  ```
  Disk does not exist
  ```



**题型2**

**创建和使用分区**

创建一个名为 ` /home/greg/ansible/partition.yml` 的playbook，它将在 `所有受管节点`上创建分区：

- 在 `vdb` 创建一个 `1500M` 主分区，分区号 `1`，并格式化 `ext4`

- `prod`组 将分区永久挂载到 `/data`

- 如果磁盘空间不够，给出提示信息

  ```
  Could not create partition of that size
  ```

  ，改为创建 `800 MiB`的分区

- 如果 `vdb` 不存在，则给出提示信息 `this disk is not exist`。

### Course

```shell
												# 题型1
# 创建分区的过程
识别磁盘 —— 分区 —— 格式化 —— 挂载


# 创建分区的帮助文档
ansible-doc parted（查询创建分区的文档）
# 格式化帮助文档
ansible-doc filesystem
# 挂载帮助文档
ansible-doc mount
# 调试模块帮助文档
ansible-doc debug         
# 设置条件帮助文档（设置条件时，建议使用is undefined，而不是使用is defined）
ansible-doc stat

# 据题创建剧本文件
vim /home/greg/ansible/partition.yml

------------------------------------
---
- name: p1
  hosts: all
  tasks:
   - block:
      - name: create 1500m
        parted:
           device: /dev/vdb
           number: 1
           state: present
           part_end: 1500MiB
      - name: Create ext4
        filesystem:
           fstype: ext4
           dev: /dev/vdb1
      - name: Mount and bind a volume
        mount:
          path: /newpart
          src: /dev/vdb1
          fstype: ext4
          state: mounted
      - name: tasks2
        block:
          - name: create vdc 1500MiB
            parted:
               device: /dev/vdc
               number: 1
               state: present
               part_end: 1500MiB
        rescue:
          - debug:
              msg: Could not create partaion of that size
            when: ansible_facts.devices.vdc is defined
          - name: create 800m
            parted:
              device: /dev/vdc
              number: 1
              state: present
              part_end: 800MiB
            when: ansible_facts.devices.vdc is defined
          - debug:
              msg: Disk does not exist
            when: ansible_facts.devices.vdc is undefined
        always:
          - name: Create ext4
            filesystem:
                fstype: ext4
                dev: /dev/vdc
          - name: Mount and bind a volume
            mount:
             path: /newpart1
             src: /dev/vdc
             fstype: ext4
------------------------------------
 
 # 运行剧本文件
 ansible-playbook partition.yml
 # 验证
 ansible all -a 'lsblk'
```

```shell
												# 题型2

# 创建分区的过程
识别磁盘 —— 分区 —— 格式化 —— 挂载


# 创建分区的帮助文档
ansible-doc parted（查询创建逻辑卷的文档）
# 格式化帮助文档
ansible-doc filesystem
# 挂载帮助文档
ansible-doc mount
# 调试模块帮助文档
ansible-doc debug         
# 设置条件帮助文档（设置条件时，建议使用is undefined，而不是使用is defined）
ansible-doc stat

# 根据题目编写剧本
vim /home/greg/ansible/partition.yml

------------------------------------
---
- name: p1
  hosts: all  # 在所有受管节点上运行
  tasks:
    - block:  # 任务块，会按照顺序执行，除非发生错误
       # 在 vdb 创建一个 1500MiB
       - name: create vdb 1500M
         parted:
            device: /dev/vdb
            number: 1  # 创建一个名为 /dev/vdb1的新分区
            state: present # 此参数表示如果不存在分区，则创建分区
            part_end: 1500MiB # 指定新分区的结束位置（1500MiB）
       
       # 将 /dev/vdb1 格式化为ext4格式
       - name: Create a ext4
         filesystem:
             fstype: ext4
             dev: /dev/vdb1
	  
	  # 挂载分区并绑定目录
       - name: Mount and bind a volume
         mount:
           path: /data # 挂载目录
           src: /dev/vdb1 # 挂载设备
           state: mounted # 挂载后立即激活分区
           fstype: ext4 # 使用 ext4 文件系统格式
         when: inventory_hostname in groups.prod # 收管主机的主机名 为 groups.prod（主机组中的设备时）执行
      
      rescue:   # 如果block任务块出现错误，执行rescue异常模块下方任务
         - debug:
             msg: Could not create partition of that size
         - name: create vdb 800M
           parted:
              device: /dev/vdb
              number: 1
              state: present
              part_end: 800MiB
           when: ansible_facts.devices.vdb is defined # 当系统上名为vdb的设备信息被找到时
         - debug:
             msg: this disk is not exist
           when: ansible_facts.devices.vdb is undefined # 当系统上名为vdb的设备信息未被找到时
 ------------------------------------
 # 运行剧本文件
 ansible-playbook partition.yml
 # 验证
 ansible all -a 'lsblk'
```

### Verify

![image-20230327181903440](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303271819610.png)

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303271856743.png" alt="image-20230327185619732" style="zoom:80%;" />



<br />



## 9. 生成主机文件

**生成主机文件**

- 将一个初始模板文件从 `http://materials/hosts.j2` 下载到 `/home/greg/ansible`
- 完成该模板，以便用它生成以下文件：针对每个清单主机包括一行内容，其格式与 `/etc/hosts`相同
- 创建名为 `/home/greg/ansible/hosts.yml` 的 playbook，它将使用此模板在 `dev` 主机组中的主机上生成文件 `/etc/myhosts`。

该 playbook 运行后， `dev` 主机组中主机上的文件 `/etc/myhosts` 应针对每个受管主机包含一行内容：

### Course

```shell
# 将 初始模板文件 下载到 指定目录
wget http://materials/hosts.j2 -P /home/greg/ansible

# 题目要求编写跟 /etc/hosts 文件一样的内容（先使用cat进行查看）
cat /etc/hosts  # 结果如下图，需要使用到魔法变量：（受控主机IP地址、域名、主机名） + for循环

# 完成模板文件
vim /home/greg/ansible/hosts.j2
-------------------------------------
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6

# groups['all']：列出清单中的所有组和主机，使用for循环，将组和主机传递给i
{% for i in groups['all']  %}
# hostvars用于引用其他主机的变量值
# 观察/etc/hosts，需要输出 ip地址 域名 主机名
{{hostvars[i].ansible_default_ipv4.address}} {{hostvars[i].ansible_fqdn}} {{hostvars[i].ansible_hostname}}
{% endfor %}
-------------------------------------

# 据题创建并完成 hosts.yml剧本文件
vim /home/greg/ansible/hosts.yml 
-------------------------------------
---
- name:
  hosts: all # 要获取收官节点主机的信息，故为all。（题目注意点）
  tasks:
    - name: Template a file
    template:
      src: /home/greg/ansible/hosts.j2
      dest: /etc/myhosts
    # group_names：包含当前主机所属的所有组的名称列表
    when: "'dev' in group_names"     # 当主机属于 "dev"组时，执行上方任务
-------------------------------------

# 运行剧本文件并验证
ansible-playbook hosts.yml 
# 验证(只在dev中生成)
ansible dev -a "cat /etc/hosts"
```

### Verify

**模板文件：**

![image-20230327201323018](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303272013829.png)

**剧本文件：**

![image-20230327201226056](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303272013570.png)

**验证：**

### ![image-20230327194446724](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303271944207.png)



<br />



## 10. 修改文件内容

**修改文件内容**

按照下方所述，创建一个名为 ` /home/greg/ansible/issue.yml` 的playbook：

- 该 playbook 将在 `所有清单主机` 上运行
- 该 playbook 会将 `/etc/issue` 的内容替换为下方所示的一行文本：
- 在 `dev` 主机组中的主机上，这行文本显示 为：`Development`
-  在 `test` 主机组中的主机上，这行文本显示 为：`Test`
-  在 `prod` 主机组中的主机上，这行文本显示 为：`Production`

### Course

```shell
# 题目要求，针对不同的受控主机替换不同的内容。使用 `copy` 模块进行复制替换（实现覆盖）
# 帮助文档：ansible-doc copy

vim /home/greg/ansible/issue.yml
-------------------------------------
--- 
- name: p1
  hosts: all
  tasks:
    - name: Copy1
      copy:
        content: 'Development'        
        dest: /etc/issue
      when: "'dev' in group_names"
      
    - name: Copy2
      copy:
        content: 'Test'        
        dest: /etc/issue
      when: "'test' in group_names"
      
    - name: Copy3
      copy:
        content: 'Production'        
        dest: /etc/issue
      when: "'prod' in group_names"
-------------------------------------
# 运行脚本并验证
ansible-playbook issue.yml
# 验证
ansible all -a "cat /etc/issue"
```

### Verify

**剧本文件：**

![image-20230316150335014](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303161503467.png)

**验证：**

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303161426685.png" alt="image-20230316142638242" style="zoom:80%;" />



<br />



## 11.  创建 Web 内容目录(注意)

**创建 Web 内容目录**

按照下方所述，创建一个名为 `/home/greg/ansible/webcontent.yml` 的playbook：

- 该 playbook 在 `dev`主机组中的受管节点上运行

- 创建符合下列要求的目录 `/webdev`：

  - 所有者为 `webdev` 组
  - 具有常规权限：`owner=read+write+execute ， group=read+write+execute ，other=read+execute`
  - 具有`特殊权限`：设置组 ID

- 用符号链接将 `/var/www/html/webdev` 链接到 `/webdev`

- 创建文件 `/webdev/index.html`，其中包含如下所示的单行文件： `Development`

- 在 `dev` 主机组中主机上浏览此目录（例如 `http://172.25.250.9/webdev/`） 将生成以下输出：

  ```
  Development
  ```

  

### Course

```shell
# 关于普通权限
owner：文件所有者 —— 4
group: 所属组 —— 2
other：其他用户 —— 1

# 关于特殊权限
SUID: 用户ID —— 4
SGID: 组ID —— 2
Sticky Bit: 粘着位 —— 1

# 首先确保 《Web服务器是否安装并开启，开启防火墙端口服务 》
ansible dev -a "rpm -q httpd"
ansible dev -a "systemctl status httpd"  # 这里发现安装了，但是并没有开启服务
ansible dev -a "grep webdev /etc/group"  # 检查 webdev 组是否存在

# 创建目录 —— 帮助文档：ansible-doc file
# 创建符号链接 —— 帮助文档：ansible-doc link
# 创建文件并包含单行内容 —— 使用copy模块 —— 帮助文档：ansible-doc copy

vim /home/greg/ansible/webcontent.yml
-------------------------------------
---
- name：webcontent
  hosts: dev
  tasks:
    # 安装httpd包，已安装则保持最新状态
    - name: install 
      yum:
        name: httpd
        state: latest
    # 启动 httpd 服务，设置开机时自动启动
     - name: start httpd service
       service:
          name: httpd
          state: started
          enabled: yes
    # 启动 firewalld 服务，并设置在开机时自动启动
     - name: open firewalld port
       service:
          name: firewalld
          state: started
          enabled: yes
   # 打开 http 服务防火墙端口，并永久启用该规则，立即应用规则变更
     - name: start firewalld service
       firewalld:
           service: http
           permanent: yes
           state: enabled
           immediate: yes
   
   # 创建目录/webdev（使用 file 模块）
     - name: Create directory
       file:
          path: /webdev # 目录的完整路径
          state: directory # 类型：目录
          mode: '2775' # 权限模式 特殊权限为SGID（组ID：2）常规权限为 rwx（4,2,5）
          group: webdev # 所属者为 webdev组（这里是设置所有组，不是所有者。所有者使用owner参数）
   
   # 创建软链接
     - name: Create a symbolic link
       file:
          src: /webdev # 源路径
          dest: /var/www/html/webdev # 目标路径（看题要注意，不要写反了，这里是将上面创建好的目录进行链接）
          state: link # 类型：链接
   # 创建文件并编写内容 
     - name: Copy using inline content
       copy:
         content: 'Development' # 写入内容
         dest: /webdev/index.html # 创建或覆盖该文件
         setype: httpd_sys_content_t # selinux安全上下文设置为httpd_sys_content_t，以允许Apache HTTP服务器访问
-------------------------------------
# 运行剧本文件
ansible-playbook webcontent.yml
# 验证
curl node1/webdev/index.html
```

### Verify

![image-20230328110025044](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303281100237.png)



<br />



## 12. 生成硬件文件

**生成硬件文件**

创建一个名为 ` /home/greg/ansible/hwreport.yml` 的playbook，它将在所有受管节点生成以下信息输出文件` /root/hwreport.txt`：

- `清单主机名称`

- 以 `MB` 表示的 `总内存大小`
- `BIOS` 版本
- 磁盘设备 `vda 的大小`
- 磁盘设备 `vdb 的大小`

- 输出文件中的每一行含有一个 `key=value` 对。

您的 playbook 应当：

- 从 `http://materials/hwreport.empty` 下载文件，并将它保存为 `/root/hwreport.txt`
- 使用`正确的值` 改为 /root/hwreport.txt
- 如果硬件项不存在，相关的值应设为 `NONE`

### Course

```shell
# 下载文件 —— 帮助文档
ansible-doc get_url
# 查找并修改某行 —— 帮助文档
ansible-doc lineinfile

# 查找主机名 魔法变量
ansible node2 -m setup -a 'filter=*host*'
# 查找内存 魔法变量
ansible node2 -m setup -a 'filter=*mem*'
# 查找BIOS 魔法变量
ansible node2 -m setup -a 'filter=*bios*'
# 查找磁盘设备 魔法变量
ansible node2 -m setup -a 'filter=*device*'


# 题目要求创建一个playbook，从链接下载文件，并将它保存为指定形式
vim /home/greg/ansible/hwreport.yml
-------------------------------------
---
- name: p1
  hosts: all # 在所有受管节点
  tasks:
    - name: Download foo.conf
      get_url:
          url: http://materials/hwreport.empty # 从 URL 下载文件
          dest: /root/hwreport.txt # 将文件到指定路径并改名

    - name: Ensure1
      lineinfile:  # 用于在文件中查找并修改某一行
          path: /root/hwreport.txt  # 文件路径
          regexp: '^HOST='  # 正则表达式
          line: HOST={{inventory_hostname | default('NONE',true)}} # 主机名 | 当找不到匹配值则返回NONE
          
    - name: Ensure2
      lineinfile:
          path: /root/hwreport.txt
          regexp: '^MEMORY='
          line: MEMORY={{ansible_memtotal_mb | default('NONE',true)}}
 
    - name: Ensure3
      lineinfile:
          path: /root/hwreport.txt
          regexp: '^BIOS='
          line: BIOS={{ansible_bios_version | default('NONE',true) }}

    - name: Ensure4
      lineinfile:
          path: /root/hwreport.txt
          regexp: '^DISK_SIZE_VDA='
          line: DISK_SIZE_VDA={{ansible_devices.vda.size | default('NONE',true)}}

    - name: Ensure5
      lineinfile:
          path: /root/hwreport.txt
          regexp: '^DISK_SIZE_VDB='
          line: DISK_SIZE_VDA={{ansible_devices.vdb.size | default('NONE',true)}}
-------------------------------------
# 运行脚本
ansible-playbook hwreport.yml
# 验证
ansible all -a 'cat /root/hwreport.txt'
```



### Verify

![image-20230328155754079](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303281557797.png)

![image-20230328155819052](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303312041092.png)



<br />

## 13. 创建密码库

**创建密码库**

按照下方所述，创建一个 `Ansible` 库来存储用户密码：

- 库名称 为 `/home/greg/ansible/locker.yml`
- 库中含有两个变量，名称如下：
  - `pw_developer`，值为 `Imadev`
  - `pw_manager`，值为 `Imamgr`
- 用于加密和解密该库的密码为 `whenyouwishuponastar`
- 密码存储在文件 `/home/greg/ansible/secret.txt` 中

### Course

```shell
# ansible-vault:管理配置工具Ansible的一项功能，它允许用户加密敏感数据

# 据题创建并编写剧本文件
vim /home/greg/ansible/locker.yml
> ---
> pw_developer: Imadev
> pw_manager: Imamgr

# 设定加密密码
ansible-vault encrypt /home/greg/ansible/locker.yml
> 输入密码：whenyouwishuponastar
> 确定密码：whenyouwishuponastar

# 将密码存储在指定文件中
vim /home/greg/ansible/secret.txt
> whenyouwishuponastar    # 注意点：存放密码时不能有多余空格

# 在ansible配置文件中指定密码文件的路径
vim ansible.cfg
> vault_password_file = /home/greg/ansible/secret.txt
-------------------------------------

# 验证  --bault-passwrod-file可以在命令帮助中查看
ansible-vault view /home/greg/ansible/locker.yml # 查看加密内容 ---> 输入正确成功能看到密码
ansible-vault view --vault-password-file=secret.txt locker.yml # 验证存放密码文件中密码是否正确
```

### Verify

<img src="https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303301525540.png" alt="image-20230330152530356" style="zoom:80%;" />



<br />



## 14.  创建用户账户

**创建用户账户**

- 从 `http://materials/user_list.yml` 下载要创建的用户的列表，并将它保存到 `/home/greg/ansible`
- 在本次考试中使用在其他位置创建的密码库 `/home/greg/ansible/locker.yml`。创建名为 `/home/greg/ansible/users.yml` 的playbook，从而按以下所述创建用户账户：
  - 职位描述为 `developer` 的用户应当：
    - 在 `dev` 和 `test` 主机组中的受管节点上创建
    - 从  `pw_developer` 变量分配密码
    - 是补充组 `devops` 的成员
  - 职位描述为 `manager` 的用户应当：
    - 在 `prod` 主机组中的受管节点上创建
    - 从 `pw_manager`变量分配密码
    - 是补充组 `opsmgr` 的成员
- 密码采用 `SHA512` 哈希格式
- 您的 playbook 应能够在本次考试中使用在其他位置创建的库密码文件 `/home/greg/ansible/secret.txt` 正常运行。

>题目提示剧本中需要关联两个变量文件，`vars_files`用于指定需要加载的变量文件列表。
>
>在 `playbook` 中，可通过引用 `locker.yml` 文件中定义的变量`pw_manager`，例如给用户`manager`分配密码和加密`SHA512`时就使用到。另一种方式是使用 `loop`，`loop` 是一个用户在 `Ansible` 中迭代执行任务的关键字，在 `playbook`中，`loop`用户循环遍历 `users` 列表中的每个用户，并将用户名称作为 `item.name` 的值传递给 `user` 模块。`item` 是 Ansible的一个关键字，用于指代 `loop` 循环过程中的当前迭代对象。（`loop` 关键字用于循环遍历一个可迭代对象，例如列表、字典等。在循环的每一次迭代中，`loop` 会将可迭代对象中的一个元素赋值给 `item`，从而实现对每个元素的处理。）
>

### Course

```bash
# 查询 创建用户账户 模块帮助
ansible-doc user

# 从链接下载 用户列表 到指定路径（下载用户文件）
wget http://materials/user_list.yml -P /home/greg/ansible


# 据题创建剧本文件
vim /home/greg/ansible/users.yml

-------------------------------------
---
- name: create user1
  hosts: dev,test # 执行任务的目标主机
  vars_files:  # 引入两个变量文件
    - locker.yml # 密钥、密码文件
    - user_list.yml # 用户列表文件
  tasks:
    - name: Ensure1
      group:
        name: devops
        state: present # 创建用户组 devops，确保此组存在
    
    # 创建用户
    - name: Add the user1
      user:
        name: "{{ item.name }}"    # 用户名称（动态引用 item 变量中的 name值）
        groups: devops
        password: "{{ pw_developer | password_hash('sha512') }}" 
      loop: "{{ users }}"  # loop用于循环遍历 users 列表中每个用户，并通过 item 来迭代指引
      when: item.job == "developer"

- name: create user2
  hosts: prod # 执行任务的目标主机
  vars_files: 
    - locker.yml
    - user_list.yml
  tasks:
    - name: Ensure2
      group:
        name: opsmgr
        state: present # 创建用户组 opsmgr，确保此组存在
    - name: Add the user2
      user:
        name: "{{item.name}}"
        groups: opsmgr
        password: "{{ pw_manager | password_hash('sha512') }}"
      loop: "{{ users }}"
      when: item.job == "manager"
-------------------------------------
# 验证 
# tail -2 ——> 显示最后2行
# /etc/passwd ——> 系统用户配置文件
ansible all -a 'tail -2 /etc/passwd'

```

### Verify 

![image-20230331093056210](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303310930878.png)

![image-20230331093402029](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303310934370.png)

![image-20230416114725983](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202304161147175.png)



<br />



## 15. 更新 Ansible 库的密钥

**更新 Ansible 库的密钥**

按照下方所述，更新现有 Ansible 库的密钥：

- 从 `http://materials/salaries.yml` 下载 Ansible 库到 `/home/greg/ansible`
- 当前的库的密码为 `insecure8sure`
- 新的库密码为 `bbs2you9527`
- 库使用 `新密码` 保持加密状态

### Course

```shell
# 从链接下载 Ansible 库 到指定路径
wget http://materials/salaries.yml -P /home/greg/ansible

# 更新当前库的密码  rekey：密钥更新
ansible-vault rekey salaries.yml
> 输入密码验证身份（旧密码）
> 需入要更新的密码（新密码）
> 再次确认（新密码）

# 验证是否更换密码
ansible-vault view salaries.yml
> 输入密码

# 验证库使用了新密码后是否保持加密状态
cat salaries.yml
```

### Verify 

![image-20230330192311377](https://raw.githubusercontent.com/zjh-jixiaolin/map_strong/main/202303301923665.png)



## 16、配置 cron 作业（增加）

**配置 cron 作业**

创建一个名为 ` /home/greg/ansible/cron.yml` 的playbook：

- 该 playbook 在 `test` 主机组中的受管节点上运行
- 配置 `cron` 作业，改作业 `每隔2分钟` 运行并执行以下命令：
  - `logger “EX200 in progress” `，以用户 `bob` 身份运行

### Course

```shell
vim /home/greg/ansible/cron.yml
-------------------------------------
---
- name: cron
  hosts: test
  tasks:
  - name: Ensure a job 
    cron:
      name: "check dirs"
      minute: "*/2"
      job: 'logger "EX200 in progress"'
      user: bob

# 执行 playbook
ansible-playbook cron.yml

# 验证
ansible test -a "grep EX200 /var/log/cron"
```



<br />



### 注意点

第`12`和第`9`题中的主机名称(ansible_hostname) 和 主机清单名称(ansible_inventory)是两个不一样的概念，是一个大坑。
