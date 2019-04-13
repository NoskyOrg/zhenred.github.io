---
title: ansible Learning
categories:                
- 教程
- ansible
date: 2019-04-07
---

## ansible的核心组件

- ansible core
- host iventory
  - 连接信息
- core modules
  - 核心模块，比如连接，yaml相关等
- custom modules
  - 用户自定义模块
- playbook 
  - yaml,jinjia2
- connect plugin
  - 连接方式插件，如ssh等

## ansible的特性

- 基于python，
  - 由 Paramiko,PyYAML,Jinjia2三个关键模块
- 部署简单，agentless
- 默认使用SSH协议：
  - 基于密钥认证
  - 在 inventory 文件中指定帐号和密码
- 主从模式：
  - master: ansible, ssh client
  - slave: ssh server
- 支持自定义模块：支持各种编程语言
- 支持 playbook
- 基于 模块 完成各种 任务

## 安装

- 使用 epel源
- 配置文件: /etc/ansible/ansible.cfg
  - 比如为 100台主机创建任务，压力会很大，
  - 此时可以配置一次的任务量，比如一次安装 10台
- invertory: /etc/ansible/hosts
  - 连接的 hosts信息
  - ansible 操作的主机，必须存在于 hosts文件中，
    - 否则提示 No hosts matched
  1. 直接使用 IP 或 主机名
  2. 使用 中括号 代表一个组
     - [webservers]
     - beta.example.org
     - 192.168.1.110
  3. 使用通配符
     - www[001:006].example.com 
     - 代表：www001.example.com ~ www006.example.com

## ansible-doc 

- plugin documentation tool
- ansible 模块文档
- -l 
  - 列出可以使用的模块
- -s
  - 查看指定模块的使用方式
  - ansible-doc -s yum
    - 查看 yum 模块的使用方式

## 命令

`ansible <host-pattern> [options]`

- -f forks
  - 启动的并发线程数
- -m module_name
  - 要使用的模块
- -a args
  - 模块特有的参数

### ansible 命令行参数

ansible 基于 ssh 连接 inventory 中的远程主机时，还可以通过参数指定其交互方式，如下：

- ansible_ssh_host
- ansible_ssh_port
- ansible_ssh_pass

### 单机操作

```bash
$ ansible 172.16.130.163 -m command -a 'date'
172.16.130.163 | CHANGED | rc=0 >>
Wed Mar 27 13:45:34 CST 2019
```

**第一行内容**

- 172.16.130.163: 代表此台主机返回信息
- CHANGED: 代表修改成功。如果不需要修改，则会显示为 OK
- rc=0: 返回值是 0

**第二行内容**

- 对应的结果

### 对 test1组 进行操作

```bash
$ ansible test1 -m command -a 'date'

172.16.130.17 | CHANGED | rc=0 >>
Wed Mar 27 21:51:48 CST 2019

172.16.130.163 | CHANGED | rc=0 >>                        
Wed Mar 27 13:54:55 CST 2019
```

### 对 hosts中不存在的主机操作

```bash
$ ansible 172.16.130.99 -m command -a 'date'

[WARNING]: Could not match supplied host pattern, ignoring: 172.16.130.99

[WARNING]: No hosts matched, nothing to do 
```

### 对 hosts文件中的所有主机操作

使用`all`代指所有主机

```bash
$ ansible all -m command -a 'date'
                 
172.16.130.17 | CHANGED | rc=0 >>
Wed Mar 27 21:59:15 CST 2019

172.16.130.100 | CHANGED | rc=0 >>        
Wed Mar 27 22:02:04 CST 2019     

172.16.130.163 | CHANGED | rc=0 >>
Wed Mar 27 14:02:23 CST 2019    
```

### 其他命令练习

```bash
$ ansible test2 -m command -a 'head -2 /etc/passwd'

172.16.130.100 | CHANGED | rc=0 >>
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
```

**注：**<br>
ansible 默认的 command 模块，无法使用 管道符`|`，重定向`>`等，可以使用 shell 模块代替

```bash
$ ansible test2 -m command -a 'tail -2 /etc/passwd | head -1'
172.16.130.100 | FAILED | rc=1 >>
tail: option used in invalid context -- 2non-zero return code

$ tail -2 /etc/passwd | head -1
geoclue:x:995:992:User for geoclue:/var/lib/geoclue:/sbin/nologin 
```

### 常见模块

`ansible-doc -s moduleName`

- command
  - 命令模块，也是默认模块，用于在远程执行命令
  - 无法 在命令中使用变量，或者管道符等，
  - 如果需要使用变量，管道符，选择 shell模块
- cron
  - state
    - 一个job创建后，默认是present，如果不需要了，修改为absent即可删除
    - present: 安装
    - absent: 移除
  - 每十分钟运行一次 `echo hello`
    - `*/10 * * * * /bin/echo hello`
    - `ansible test2 -m cron -a "minute='*/10' job='/bin/echo hello' name='test cron'"`
    - 没有指定的时间单位，默认为`*`
    - 一定要给一个名字`name`，否则会出错，也可以作为标记，
      - 比如`crontab -l`会显示是ansible name，方便辩论
- user
  - 系统用户管理
- group
  - 系统组管理
- copy
  - 将文件复制到远程主机
  - 如果远程主机上，已经存在同名文件，会覆盖替换
  - src: 本地源文件，可以是相对路径，或绝对路径
  - dest: 目标主机的绝对路径
  - 可以指定属主，属组，权限等
  - 可以给远程文件输入内容
    - 如果文件不存在，会新建文件，输入内容
    - content
  - 注意：content 替代了 src，两者不要同时使用
- file
  - 设定文件或目录的属性
    - 属主，属组，权限等
  - path: 指定文件路径
    - 可以使用 name 或 dest 替换
  - 创建文件的软链接
    - src: 源文件
    - path: 符号连接路径
- ping
  - 测试远程主机连通性
- service
  - 控制服务的状态
  - enabled 
    - 是否自动启动，取值为 true or false
    - runlevel 在哪个级别自动启动
  - state 
    - 查看服务的状态
    - started, stopped, restarted
- shell
  - 与 command 模块相似
  - 可以使用变量，可以使用管道符
- script
  - 将本地的脚本，发送到远程主机上，然后执行
  - 相对路径 or 绝对路径
- yum
  - 安装程序包
  - name: 包名，
    - 也可指定版本，name-1.0
    - 默认新版
  - state
    - 默认安装
      - present or latest
    - absent：卸载
  - `ansible a -m yum -a 'name=tree'`
  - `ansible a -m yum -a 'name=tree state=absent'`
- setup
  - `ansible a -m yum -a 'name=tree'`
  - 收集远程主机的 facts 信息
  - ansible 在管理主机，运行命令之前，会收集各种可能用到的信息，
    - 比如 SSH，系统信息，网卡，CPU核数性能 等
  - 本地的 ansible 可以根据收集的信息来决定下一步怎么做
    - 比如根据CPU核数，决定并发数

#### 命令练习

- cron module

```bash
$ ansible test2 -m cron -a "minute='*/10' job='/bin/echo hello' name='test cron'"
172.16.130.100 | CHANGED => {
    "changed": true,
    "envs": [],
    "jobs": [
        "test cron"
    ]
}

$ ansible test2 -a 'crontab -l'
172.16.130.100 | CHANGED | rc=0 >>
# Ansible: test cron
*/10 * * * * /bin/echo hello

$ ansible test2 -m cron -a "minute='*/10' job='/bin/echo hello' name='test cron' state=absent"
172.16.130.100 | CHANGED => {
    "changed": true,         
    "envs": [],          
    "jobs": []                    
}

$ ansible test2 -a 'crontab -l'
172.16.130.100 | CHANGED | rc=0 >> 
```

- user module

```bash
#批量创建用户
$ ansible all -m user -a 'name="user1"' 

#批量删除用户
$ ansible all -m user -a 'name="user1" state=absent' 

# 当用户的家目录存在时
$ ansible test2 -m user -a 'name=user1 shell=/bin/sh system=yes'
172.16.130.100 | CHANGED => {
    "changed": true, 
    "comment": "", 
    "create_home": true, 
    "group": 992, 
    "home": "/home/user1", 
    "name": "user1", 
    "shell": "/bin/sh", 
    "state": "present", 
    "stderr": "useradd: warning: the home directory already exists.\nNot copying any file from skel directory into it.\n", 
    "stderr_lines": [
        "useradd: warning: the home directory already exists.", 
        "Not copying any file from skel directory into it."
    ], 
    "system": true, 
    "uid": 996
}

$ ansible test2 -m user -a 'name=user1 shell=/bin/sh system=yes'                             
172.16.130.100 | CHANGED => {
    "changed": true, 
    "comment": "", 
    "create_home": true, 
    "group": 992, 
    "home": "/home/user1", 
    "name": "user1", 
    "shell": "/bin/sh", 
    "state": "present", 
    "system": true, 
    "uid": 996
}

$ ansible test2 -m user -a 'name=user1 shell=/bin/sh system=yes state=absent'

$ ansible test2 -m user -a 'name=user2 shell=/bin/bash system=yes group=root'
```

- group module

```bash
ansible test2 -m group -a 'name=group1 system=yes gid=555'                               
172.16.130.100 | CHANGED => {
    "changed": true,
    "gid": 555,
    "name": "group1",
    "state": "present",        
    "system": true
}  
```

- copy

```bash
$ ansible test2 -m copy -a 'src=/etc/passwd dest=/tmp/test.passwd owner=root mode=600'       
172.16.130.100 | CHANGED => {
    "changed": true,
    "checksum": "99d1d5c1344f3317bc093b2ec5b5da74e9b4eb41",
    "dest": "/tmp/test.passwd",
    "gid": 0,
    "group": "root",
    "md5sum": "af33f7396b5796666b2e3c42e4ee10b4",
    "mode": "0600",
    "owner": "root",
    "size": 1504,
    "src": "/root/.ansible/tmp/ansible-tmp-1553787100.84-129000440860281/source",
    "state": "file",        
    "uid": 0      
}

# 在对应主机上验证文件
$ ll /tmp/test.passwd   
-rw------- 1 root root 1504 Mar 28 23:33 /tmp/test.passwd

# 往远程主机的文件输入内容，如果没有会生成文件，注意 换行符 \n
$ ansible test2 -m copy -a 'content="hello\nhi" dest=/tmp/test.passwd'
172.16.130.100 | CHANGED => {
    "changed": true,
    "checksum": "722d79e714d35a03d9d730f655132341dadb0b3c", 
    "dest": "/tmp/test.passwd",
    "gid": 0,
    "group": "root",
    "md5sum": "14c703df76f3bd767778b402a27be15f",
    "mode": "0600",
    "owner": "root",     
    "size": 8,
    "src": "/root/.ansible/tmp/ansible-tmp-1553787530.65-272591452022139/source",
    "state": "file",       
    "uid": 0
}

# 验证文件内容，注意 hi 后面没有换行
$ cat /tmp/test.passwd 
hello
hi[root@zhenlee-dbscale-web-replica-3 ~] #
```

- file module

```bash
$ ansible test2 -m file -a 'owner=user2 group=mysql mode=707 path=/tmp/t.passwd'
172.16.130.100 | SUCCESS => {
    "changed": false,      
    "gid": 1002, 
    "group": "mysql",   
    "mode": "0707", 
    "owner": "user2",
    "path": "/tmp/t.passwd",        
    "size": 8,  
    "state": "file",        
    "uid": 995                  
}

# 验证文件
$ ll /tmp/t.passwd      
-rwx---rwx 1 user2 mysql 8 Mar 28 23:44 /tmp/t.passwd

# 创建软链接
$ ansible test2 -m file -a 'path=/tmp/t.passwd.lnk src=/tmp/t.passwd state=link'
172.16.130.100 | CHANGED => {
    "changed": true,
    "dest": "/tmp/t.passwd.lnk", 
    "gid": 0, 
    "group": "root", 
    "mode": "0777", 
    "owner": "root", 
    "size": 13, 
    "src": "/tmp/t.passwd", 
    "state": "link", 
    "uid": 0
}

# 验证
$ ll /tmp/t.* 
-rwx---rwx 1 user2 mysql  8 Mar 28 23:44 /tmp/t.passwd
lrwxrwxrwx 1 root  root  13 Mar 28 23:55 /tmp/t.passwd.lnk -> /tmp/t.passwd
```

- service module

```bash
# 控制 httpd 是否开机自启动
$ ansible 163 -a 'chkconfig --list httpd'      
172.16.130.163 | CHANGED | rc=0 >>
httpd           0:off   1:off   2:off   3:off   4:off   5:off   6:off             

$ ansible 163 -m service -a 'enabled=true name=httpd state=started'
172.16.130.163 | CHANGED => {    
    "changed": true,        
    "enabled": true,        
    "name": "httpd",      
    "state": "started"
}

$ ansible 163 -a 'chkconfig --list httpd'
172.16.130.163 | CHANGED | rc=0 >>
httpd           0:off   1:off   2:on    3:on    4:on    5:on    6:off   
```

- shell module

```bash
# 如果用默认的 command，无法使用管道符，此时验证得到密码没有被正确修改，而且也没有提示修改成功
$ ansible a -a 'echo aa | passwd --stdin user1'
172.16.130.100 | CHANGED | rc=0 >>
aa | passwd --stdin user1

# 使用 shell，可以正确识别管道符，正确修改密码
$ ansible a -m shell -a 'echo bb | passwd --stdin user1'
172.16.130.100 | CHANGED | rc=0 >>
Changing password for user user1.           
passwd: all authentication tokens updated successfully.
```

- script module

```bash
$ ansible a -m script -a '/etc/ansible/test.sh'
$ ansible a -m script -a 'test.sh'
172.16.130.100 | CHANGED => {    
    "changed": true,
    "rc": 0,       
    "stderr": "Shared connection to 172.16.130.100 closed.\r\n",        
    "stderr_lines": [ 
        "Shared connection to 172.16.130.100 closed."
    ], 
    "stdout": "",      
    "stdout_lines": []
} 
```

- setup module

下面的主机，在 ansible_facts 下面的一些变量，就是 setup 的 facts 变量，<br>
比如 ansible_all_ipv4_addresses

```bash
ansible a -m setup
172.16.130.100 | SUCCESS => {  
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [       
            "172.17.0.1",       
            "172.16.130.100",        
            "172.16.130.170"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::f816:3eff:feab:c432"  
        ],   
        "ansible_apparmor": {
            "status": "disabled"        
        },
        ...
        # 以下省略了很多信息
        ...
```

## YAML

同一个数据有多个值，叫做序列，使用 `-` 分隔<br>
和python类似，注意缩进

键值对也可以看作是映射

如下，a 的配偶是 b，<br>
b也有名字和年龄，只是 spouse 作为键，name age 都是值，同时也是键<br>
而他们的孩子有两个，使用 `-`分隔，

```yml
name: a
age: 4
spouse:
  name: b
  age: 3
children:
  -  name: c
    age: 2
  -  name: d
    age: 1

# 也可以把字典元素放在大括号中
{name: abc, age: 2, port: 22}
```

## playbooks

playbooks 就是一个或多个 play 组成的列表，play 的主要功能在于将事先归并为一组的主机装扮成事先通过 ansible 的 task 定义好的角色。

从根本上讲，所谓 task 无非是调用 ansible 的一个 module，将多个 play 组织在一个 playbook 中，即可让它们联合起来，按事先编排的机制唱戏

```yml
- hosts: a
  # a主机，或者 a组内的所有主机
  vars:
  # 定义变量，比如下面的 http_port，就可以在后面引用
    http_port: 80
    max_clients: 256
  remote_user: root
  tasks:
  # 第一个任务 task
  - name: install apache
    yum: name=httpd state=latest
  # 第二个任务 
  - name: ensure apache is running
    service: name=httpd atate=started
  handlers: 
    - name: restart apache
      service: name=httpd state=restarted
```

### playbooks 组成

```yml
inventory: 定义的主机
modules: 模块
ad hoc commands: 命令
playbooks:
    tasks: 任务，即调用模块完成的操作，多个任务用 - 隔开
    variables: 变量
    templates: 模板
    handlers: 处理器，由某事件触发执行的操作
    roles: 角色
```

### playbook 基础组件

- hosts users

playbook 中的每一个 play 的目的，都是为了让某些主机以某个指定的用户身份执行任务，

hosts 用于指定要执行任务的主机，其可以是一个或多个以`:`分隔的主机组，

remote_user 则用于指定远程主机上的执行任务的用户

```yml
- host: a
  remote_user: mysql
```

remote_user 也可用于各 task 中，也可以通过指定其通过 sudo 的方式在远程主机上执行任务，其可用于 play 全局或某任务，<br>
此外，甚至可以在 sudo 时使用 sudo_user 指定 sudo 时切换的用户

```yml
- host: a
  remote_user: mysql
  tasks:
    - name: test connection
      ping:
      remote_user: ora
      sudo: yes
      # 代表以 ora 用户，sudo 到 root 执行
    - name: second task
```

### 基本结构

```yml
- hosts: a
  remote_user:
  tasks:
    - task1
      module_name: module_args
    - task2
- hosts: b
```

### 执行顺序

以任务为中心，不是以主机为中心：<br>
a组里面的所有主机，完成第一个任务<br>
然后 a组里面的所有主机，完成第二个任务<br>
等到 a组里面的所有主机执行完毕后，b组里面的主机开始执行任务

### 任务列表和action

task list 中的各任务按次序逐个在 hosts 中的所有主机上执行，如果中途发生错误，所有已执行的任务，可能回滚。此时，更正playbook后，重新执行一次即可。因为具有幂等性

task 的目的是使用指定的参数执行模块，而在模块参数中可以使用变量，模块执行是幂等的，所以多次执行结果一致，是安全的

每个 task 都应该有 name，用于 playbook 的执行结果输出，如果没有提供 name，则 action 的结果将用于输出

定义 task 的可以使用 `action: module options` or `module: option`，推荐`后者`，以保证向后兼容，前者在较新版本中，可能无法使用。如果 action 一行的内容过多，也可以在行首使用几个空白字符进行换行。

如下，代表以`name=httpd state=running`参数，运行`service`模块

```yml
tasks:
  - name: test apache is running
    service: name=httpd state=running
```

在所有的模块中，只有 command 和 shell 模块仅需要给定一个列表而无需使用 key-value 格式，如下，不需要使用 name='/sbin/setenforce 0'

```yml
tasks:
  - name: disable selinux
    command: /sbin/setenforce 0
```

如果命令或脚本的退出码不为 0，可以使用如下方式替代：<br>
`cmd || /bin/true` or `ignore_errors` 忽略错误

一般用于，当某个操作失败，并不影响后面的任务时，可以忽略错误，防止 ansible 进程退出

```yml
tasks:
  - name: run this cmd and ignore the result
    shell: comeCMD || /bin/true

or

tasks:
  - name: run this cmd and ignore the result
    shell: comeCMD
    ignore_errors: True
```

## ansible-playbook

如果使用 yaml 配置文件，需要使用 ansible-playbook

### 实例 1

```yml
more a.yml                              
- hosts: a 
  remote_user: root              
  tasks:
  - name: create ax group
    group: name=ax system=yes gid=628
  - name: create ax user                  
    user: name=ax uid=628 group=ax system=yes

- hosts: a
  remote_user: root              
  tasks:
  - name: copy file to a                  
    copy: src=/etc/passwd dest=/tmp/passwd.test
```

#### 结果验证

- 从以下输出内容也可以看出，
- ansible会按顺序先开始 play a，之后收集 facts 信息
- 等到第一个任务在所有主机上都处理完，再处理第二个任务。
- 之后开始接下来的 play

**注意**

这里由于是第一次执行，在创建用户的时候，相当于改动了内容，所以除了输出`OK`，还会输出`changed`<br>
由于幂等性，再次执行这个命令，由于用户已经存在，不会重复创建，只会输出`OK`，不会输出`changed`

```bash
$ ansible-playbook a.yml

PLAY [a] **************************************************************************************************************************************         

TASK [Gathering Facts] ************************************************************************************************************************
ok: [172.16.130.17]        
ok: [172.16.130.100]     

TASK [create ax group] ************************************************************************************************************************
changed: [172.16.130.17]   
changed: [172.16.130.100]    

TASK [create ax user] *************************************************************************************************************************
changed: [172.16.130.17]   
changed: [172.16.130.100]    

PLAY [a] **************************************************************************************************************************************         

TASK [Gathering Facts] ************************************************************************************************************************
ok: [172.16.130.17]        
ok: [172.16.130.100]    

TASK [copy file to a] *************************************************************************************************************************
changed: [172.16.130.17]   
changed: [172.16.130.100]  

PLAY RECAP ************************************************************************************************************************************
172.16.130.100             : ok=5    changed=3    unreachable=0    failed=0
172.16.130.17              : ok=5    changed=3    unreachable=0    failed=0
```

**注意**

此时，再执行一次上面的命令，验证幂等性。只会输出`OK`，不会输出`changed`，因为没有内容要修改

```bash
$ ansible-playbook a.yml

PLAY [a] **************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]                            
        
TASK [create ax group] ************************************************************************************************************************
ok: [172.16.130.17]                            
ok: [172.16.130.100]                            
        
TASK [create ax user] *************************************************************************************************************************
ok: [172.16.130.17]                            
ok: [172.16.130.100]                            
        
PLAY [a] **************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************
ok: [172.16.130.17]                       
ok: [172.16.130.100]

TASK [copy file to b] *************************************************************************************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

PLAY RECAP ************************************************************************************************************************************
172.16.130.100             : ok=5    changed=0    unreachable=0    failed=0   
172.16.130.17              : ok=5    changed=0    unreachable=0    failed=0   
```

### 实例 2

```yml                      
- hosts: a
  remote_user: root
  tasks:
  - name: install httpd
    yum: name=httpd state=latest
  - name: install cnf file for httpd
    copy: src=/tmp/httpd.conf dest=/tmp/httpd.conf
  - name: start httpd service
    service: enabled=true name=httpd state=started
```

#### 结果验证

```bash
$ ansible-playbook apache.yml   
PLAY [a] ***********************************************************************                                                          
TASK [Gathering Facts] *********************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]      

TASK [install httpd] ***********************************************************
ok: [172.16.130.17]  
ok: [172.16.130.100]                                                 
TASK [install cnf file for httpd] **********************************************                 
changed: [172.16.130.17]
changed: [172.16.130.100]      

TASK [start httpd service] *****************************************************
changed: [172.16.130.17]
changed: [172.16.130.100]      

PLAY RECAP *********************************************************************
172.16.130.100             : ok=4    changed=2    unreachable=0    failed=0
172.16.130.17              : ok=4    changed=2    unreachable=0    failed=0
```

---

在上面的实例2中，第一步安装 httpd，然后上传配置文件，之后启动服务

但是由于幂等性，造成的结果是：

1. 软件已经安装，不会再次安装，没问题，
2. 如果修改配置文件，会检测到文件不同，然后传输过去，覆盖旧文件，没问题，
3. 检测到服务已经启动了，不会再次启动。这就有问题了，因为修改配置文件要重启服务

此时就需要 handler 了

## handler 处理器

handler 会监控某个 task，用于当关注的资源发生变化时采取一定的操作

notify 某个 action，可用于在每个 Play 的最后被触发，在最后执行一次，这样可以避免多次有改变发生时，每次都执行指定的操作。在 notify 中列出的操作称为 handler，也就是 notify 中调用 handler 中定义的操作。

如下：<br>
当程序检测到 tem.cnf 改变了时，就会被发现，之后通知 handlers，执行对应的操作

notify 里面的对应的操作，是 handlers 里面，对应的 tasks 的 name

```yml
- name: template cnf file
  template: src=tem.cnf dest=/tmp/tem.cnf
  notify:
    - restart memcached
    - restart apache
  
  handlers:
  - name: restart memcached
    service: name=memcached state=restarted
  - name: restart apache
    service: name=httpd state=restart
```

### 实例 1

```yml
- hosts: a
  remote_user: root
  tasks:
  - name: install httpd
    yum: name=httpd state=latest
  - name: install cnf file for httpd
    copy: src=/tmp/httpd.conf dest=/tmp/httpd.conf
    notify:
    - restart httpd
    - echo yes
  - name: start httpd service
    service: enabled=true name=httpd state=started
  handlers:
  - name: restart httpd
    service: name=httpd state=restarted
  - name: echo yes
    shell: echo yes > /tmp/yes
```

## 变量 vars

上面的实例1 添加 `vars`，使用变量。<br>
如果需要引用变量，使用 两层大括号 `{{varsName}}`

在 playbook 中，不一定必须引用 vars 中定义的变量，也可以引用 ansible 中定义的所有变量。都可以使用`{{varsName}}`引用<br>
比如使用 `ansible a -m setup` 中，facts 中的信息

也可以直接向主机传递一些变量，比如在 hosts 文件中，为 100主机添加一个变量 `testVar='centOS 7'`，为 17主机添加变量 `testVar='centOS 6`

```txt
[varTest]
172.16.130.100 testVar='centOS 7'
172.16.130.17 testVar='centOS 6'
```

如果一个组内的变量一致，也可以在 /etc/ansible/hosts 中使用组变量

```bash
[db]
10.0.0.147 aa=abc

[db:vars]
ansible_ssh_user=root
ansible_ssh_pass=abc123 
port=3306
```

### 实例1

```yml
- hosts: a
  remote_user: root
  vars:
  - package: httpd
  - service: httpd
  - echoTest: "this is test letters"
  tasks:
  - name: install httpd
    yum: name={{package}} state=latest
  - name: install cnf file for httpd
    copy: src=/tmp/httpd.conf dest=/tmp/httpd.conf
    notify:
    - restart httpd
    - echo yes
  - name: start httpd service
    service: enabled=true name={{service}} state=started
  handlers:
  - name: restart httpd
    service: name=httpd state=restarted
  - name: echo yes
    shell: echo {{echoTest}} > /tmp/yes
```

### 实例2

比如变量使用 facts 中的 `ansible_all_ipv4_addresses`

```yml
- hosts: a
  remote_user: root
  tasks:
  - name: copy file
    copy: content={{ansible_all_ipv4_addresses}} dest=/tmp/aHosts
```

#### 结果验证

```bash
cat /tmp/aHosts                           
["172.16.130.17"]
```

### 实例3

也可以使用 hosts 文件中定义的变量

```yml
- hosts: varTest
  remote_user: root
  tasks:
  - name: copy file
    copy: content='{{ansible_all_ipv4_addresses}},{{testVar}}' dest=/tmp/varTest
```

#### 结果验证

```bash
$ cat /tmp/varTest
[u'172.16.130.17'],centOS 6

# 另一台主机有多个IP，是由于有虚拟IP，
$ cat /tmp/varTest
[u'172.17.0.1', u'172.16.130.100', u'172.16.130.170'],centOS 7
```

## when 条件测试

如果需要根据变量，facts或此前任务的执行结果来作为某 task 执行与否的前提时要用到条件测试

### when

在 task 后添加 when 子句即可使用条件测试，when 语句支持 Jinjia2 表达式语法，例如: 只关闭 Debian 的主机

```yml
tasks:
  - name: 'shutdown debian flavored systems'
    command: /sbin/shutdown -h now
    when: ansible_os_family == 'Debian'
```

### 实例1

当主机名为 `zhenlee-dbscale-web-replica-3.novalocal` 时，创建用户 user10

```yml
- hosts: a
  remote_user: root
  vars:
  - username: user10
  tasks:
  - name: create {{username}} user
    user: name={{username}}
    when: ansible_fqdn == 'zhenlee-dbscale-web-replica-3.novalocal'
```

其中，ansible_fqdn 是 `ansible a -m setup` 中的主机名

#### 结果验证

从输出可以看到，当主机名不匹配时，比如 172.16.130.17，会显示为 `skipping: [172.16.130.17]`

```bash
ansible-playbook a.yml

PLAY [a] ***********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

TASK [create user10 user] ******************************************************
skipping: [172.16.130.17]
changed: [172.16.130.100]

PLAY RECAP *********************************************************************
172.16.130.100             : ok=2    changed=1    unreachable=0    failed=0   
172.16.130.17              : ok=1    changed=0    unreachable=0    failed=0   
```

## 迭代

当有需要重复执行的任务时，可以使用迭代机制，其使用格式为将需要迭代的内容定义为 item 变量引用，并通过 with_items 语句来指明迭代的元素列表即可

- 迭代、重复同类 task 时使用
  - 调用：item
  - 定义列表：with_items
    - 列表值 也可以是字典，但引用时，要使用 `item.KEY` 形式 
- ansible 的循环机制还有更多的高级功能，<docs.ansible.com/playbook_loops.html>

例如:<br>
其中的`{{ item }}`是固定的变量名，代表的是：重复的引用`with_items`列表中的内容，直到列表中的内容遍历结束

```yml
- name: add several users
  user: name={{item}} state=present groups=wheel
  with_items:
    - testuser1
    - testuser2
```

with_item 中可以使用的元素还可为 hashes。意味着，每个元素都还可以有自己的值。

例如：<br>

- item.names 代表 列表中的 name 对应的值，
- item.groups 代表 列表中的 group 对应的值

其中 `item.` 是固定的前缀格式，后面的 键值 自己定义

```yml
- name: add several users
  user: name={{item.names}} state=present groups={{item.groups}}
  with_items:
    - {name: 'testuser1', group: 'wheel'}
    - {name: 'testuser2', group: 'root'}

# 比如
user: name={{item.appName}} state=start
groups={{item.appConf}}
with_items:
- {appName: apache, appConf: /tmp/httpd.conf}
- {appName: php, appConf: /tmp/php.ini}
- {appName: mysql, appConf: /tmp/my.cnf}
```

## templates 模板

```bash
$ cat templates/port.cnf
Listen {{http_port}}
MaxClients {{maxClients}}
ServerName {{ansible_fqdn}} 

$ cat hosts
[varTest4]
172.16.130.100 http_port=12 maxClients=34
172.16.130.17 http_port=56 maxClients=78
```

### 实例

```yml
- hosts: a
  remote_user: root          
  vars:
  - package: httpd
  - service: httpd
  - echoTest: "this is test letters"         
  tasks:
  - name: install httpd
    yum: name={{package}} state=latest
  - name: install cnf file for httpd
    # copy: src=/tmp/httpd.conf dest=/tmp/httpd.conf
    template: src=/etc/ansible/templates/port.cnf dest=/tmp/port.cnf
    notify:
    - restart httpd     
    - echo yes
  - name: start httpd service      
    service: enabled=true name={{service}} state=started           
  handlers:
  - name: restart httpd
    service: name=httpd state=restarted 
  - name: echo yes
    shell: echo {{echoTest}} > /tmp/yes
```

#### 结果验证

```bash
$ cat /tmp/port.cnf
Listen 56  
MaxClients 78
ServerName wangxianran-s6000-2.novalocal

$ cat /tmp/port.cnf
Listen 12  
MaxClients 34
ServerName zhenlee-dbscale-web-replica-3.novalocal 
```

## 算术运算

## 标签 tags

有时只想运行某个 task，而不是运行全部的任务，此时可以使用 标签。运行时加上 `--tags='tagName'`

在 playbook 可以为某个或某些任务定义一个 标签，在执行此 playbook 时，通过为 `ansible-playbook` 命令使用 `--tags` 选项能实现仅运行指定的 tasks 而非所有的，

特殊 tags：always。代表无论使用哪个 tag，always 总会触发

### 实例1

```yml
- hosts: a
  remote_user: root          
  vars:
  - package: httpd
  - service: httpd
  - echoTest: "this is test letters"         
  tasks:
  - name: install httpd
    yum: name={{package}} state=latest
  - name: install cnf file for httpd
    # copy: src=/tmp/httpd.conf dest=/tmp/httpd.conf
    template: src=/etc/ansible/templates/port.cnf dest=/tmp/port.cnf
    tags:
    - cnf
    # 添加一个名字为 cnf 的 tag
    notify:
    - restart httpd     
    - echo yes
  - name: start httpd service      
    service: enabled=true name={{service}} state=started           
  handlers:
  - name: restart httpd
    service: name=httpd state=restarted 
  - name: echo yes
    shell: echo {{echoTest}} > /tmp/yes
```

#### 结果验证

可以看到，添加 `--tags='cnf'` 后，只执行了 传输文件 和 被触发的 when 任务

```bash
$ ansible-playbook apache.yml --tags='cnf' 

PLAY [a] ***********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

TASK [install cnf file for httpd] **********************************************
changed: [172.16.130.17]                
changed: [172.16.130.100]

RUNNING HANDLER [restart httpd] ************************************************
changed: [172.16.130.17]                
changed: [172.16.130.100]

RUNNING HANDLER [echo yes] *****************************************************
changed: [172.16.130.17]
changed: [172.16.130.100]

PLAY RECAP *********************************************************************
172.16.130.100             : ok=4    changed=3    unreachable=0    failed=0   
172.16.130.17              : ok=4    changed=3    unreachable=0    failed=0
```

### 实例2

添加特殊 tags `always`，代表始终执行

```yml
- hosts: a
  remote_user: root          
  vars:
  - package: httpd
  - service: httpd
  - echoTest: "this is test letters"         
  tasks:
  - name: install httpd
    yum: name={{package}} state=latest
    tags:
    - always
  - name: install cnf file for httpd
    # copy: src=/tmp/httpd.conf dest=/tmp/httpd.conf
    template: src=/etc/ansible/templates/port.cnf dest=/tmp/port.cnf
    tags:
    - cnf
    # 添加一个名字为 cnf 的 tag
    notify:
    - restart httpd     
    - echo yes
  - name: start httpd service      
    service: enabled=true name={{service}} state=started   
    tags:
    - service        
  handlers:
  - name: restart httpd
    service: name=httpd state=restarted 
  - name: echo yes
    shell: echo {{echoTest}} > /tmp/yes
```

#### 结果验证

```bash
$ ansible-playbook  apache.yml --tags='cnf'

PLAY [a] *******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

TASK [install httpd] *******************************************************************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

TASK [install cnf file for httpd] ******************************************************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

PLAY RECAP *****************************************************************************************************************
172.16.130.100             : ok=3    changed=0    unreachable=0    failed=0
172.16.130.17              : ok=3    changed=0    unreachable=0    failed=0

$ ansible-playbook  apache.yml --tags='service'

PLAY [a] *******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

TASK [install httpd] *******************************************************************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

TASK [start httpd service] *************************************************************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

PLAY RECAP *****************************************************************************************************************
172.16.130.100             : ok=3    changed=0    unreachable=0    failed=0
172.16.130.17              : ok=3    changed=0    unreachable=0    failed=0
```

## roles

定义的格式可以反复使用

1. 目录名同角色名
2. 目录结构有固定格式
   - 以下用不到的目录可以创建为空目录，也可以不创建
   - files
     - 静态文件
   - templates
     - Jinjia2模板文件
     - 如果调用 files 里面的文件，可以直接写文件的名字
   - tasks:
     - 至少有一个 main.yml 文件，定义各 tasks
   - handlers:
     - 至少有一个 main.yml 文件，定义各 handlers
   - vars:
     - 至少有一个 main.yml 文件，定义变量
   - meta:
     - 定义依赖关系等信息
3. site.yml 中定义 playbook，额外也可以有其它的 yml 文件


### 创建 role 的步骤

1. 创建以 roles 命名的目录
2. 在 roles 目录中分别创建以各角色名称命名的目录，如 testA
3. 在每个角色命名的目录中分别创建 files,handlers,meta,tasks,templates,vars 目录
   - 用不到的目录可以创建为空目录，也可以不创建
4. 在 playbook 文件中，调用各角色

### role 内各目录中可用的文件

- tasks日录:
  - 至少应该包含一个名为main.yml的文件，
  - 其定义了此角色的任务列表:
    - 此文件可以使用include包含其它的位于此目录中的task文件
- flles目录:
  - 存放由copy或script等模块调用的文件:
- templates目录：
  - template模块会 白动在此目录中找Jinja2模板文件:
- handlers目录：
  - 此目录中应当包含一个main.yml文件，用于定义此角色用到的各handler:
    - 在handler中 使用inc lude包含的其它的handler文件也应该位于此目录中
- vars目录：
  - 应当包含一个main.ynl文件，用于定义此角色用到的变量
- meta目录：
  - 应当包含一个main.ynl文件，用于定义此角色的特殊设定及其依赖关系
  - ansible 1.3 及其以后的版本才支持，
- default日录:
  - 为当前角色设定默认交量时使用此目录:应当包含-个main.yml文件

### 使用方式

```yml
# 直接使用
- hosts: a
  roles: 
    - webservers
    - databases

# 向 role 传递参数
- hosts: a
  roles:
    - webservers
    - {role: databases, dir: '/usr/local/mysql', port: 3306}
    - {role: databases, dir: '/usr/local/mariadb', port: 3307}

# 条件式地使用 roles
- hosts: a
  roles:
    - {role: someRole, when: "ansible_os_family == 'RedHat'"}

# role dependencies
# role 也可以指定依赖的其他 role
# 可以查看官方文档，一般使用不到
```

### 实例1

```bash
$ mkdir -pv ansible_playbooks/roles/{webservers,databases}/{files,handlers,meta,tasks,templates,vars}

# 添加相关配置文件
$ tree ansible_playbooks/
ansible_playbooks/
├── main.yml
├── roles
│   ├── databases
│   │   ├── files
│   │   │   └── my.cnf
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   └── vars
│   └── webservers
│       ├── files
│       │   └── httpd.conf
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       └── vars
└── site.yml
```

### webservers 文本内容

```yml
$ cat site.yml 
- hosts: a
  remote_user: root
  roles:
  - webservers

$ cat files/httpd.conf 
this is test

$ cat tasks/main.yml
- name: install httpd package
  yum: name=httpd
- name: install cnf file
  copy: src=httpd.conf dest=/tmp/roleTest.cnf
  # 注意：看这里的 src源文件，直接使用的相对路径
  tags:
  - conf
  notify:
  - restart httpd
- name: start httpd
  service: name=httpd state=started

$ cat handlers/main.yml
- name: restart httpd
  service: name=httpd state=restarted
```

### databases 文本内容

```yml
$ cat files/my.cnf        
aaa

$ cat tasks/main.yml 
- name: install mysql
  yum: name=mysql-server
  ignore_errors: True
- name: install mariadb
  yum: name=mariadb-server
  ignore_errors: True
- name: install cnf file
  copy: src=my.cnf dest=/tmp/
  tags:
  - mycnf
  notify:
  - restart mysqld
  - restart mariadb
- name: start mysqld
  service: name=mysqld enabled=true state=started
  ignore_errors: True
- name: start mariadb
  service: name=mariadb enabled=true state=started
  ignore_errors: True

$ cat handlers/main.yml 
- name: restart mysqld
  service: name=mysqld state=restarted
  ignore_errors: True
- name: restart mariadb
  service: name=mariadb state=restarted
  ignore_errors: True
```

#### 结果验证

```bash
$ ansible-playbook site.yml     
PLAY [a] ***********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

TASK [webservers : install httpd package] **************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

TASK [webservers : install cnf file] *******************************************
changed: [172.16.130.100]
changed: [172.16.130.17]

TASK [webservers : start httpd] ************************************************
ok: [172.16.130.17]
ok: [172.16.130.100]

RUNNING HANDLER [webservers : restart httpd] ***********************************
changed: [172.16.130.17]
changed: [172.16.130.100]

PLAY RECAP *********************************************************************
172.16.130.100             : ok=5    changed=2    unreachable=0    failed=0   
172.16.130.17              : ok=5    changed=2    unreachable=0    failed=0   
```

### 实例2

```yml
$ cat site.yml

- hosts: 172.16.130.17
  remote_user: root
  roles:
  - webservers

- hosts: 172.16.130.100
  remote_user: root
  roles:
  - databases

- hosts: 172.16.130.61
  remote_user: root
  roles:
  - webservers
  - databases
```

#### 结果验证

```bash
$  ansible-playbook site.yml 

PLAY [172.16.130.17] ***********************************************************

TASK [Gathering Facts] *********************************************************
ok: [172.16.130.17]

TASK [webservers : install httpd package] **************************************
ok: [172.16.130.17]

TASK [webservers : install cnf file] *******************************************
ok: [172.16.130.17]

TASK [webservers : start httpd] ************************************************
ok: [172.16.130.17]

PLAY [172.16.130.100] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [172.16.130.100]

TASK [databases : install mysql] ***********************************************
fatal: [172.16.130.100]: FAILED! => {"changed": false, "msg": "No package matching 'mysql-server' found available, installed or updated", "rc": 126, "results": ["No package matching 'mysql-server' found available, installed or updated"]}
...ignoring

TASK [databases : install mariadb] *********************************************
ok: [172.16.130.100]

TASK [databases : install cnf file] ********************************************
ok: [172.16.130.100]

TASK [databases : start mysqld] ************************************************
fatal: [172.16.130.100]: FAILED! => {"changed": false, "msg": "Could not find the requested service mysqld: host"}
...ignoring

TASK [databases : start mariadb] ***********************************************
ok: [172.16.130.100]

PLAY [172.16.130.61] ***********************************************************

TASK [Gathering Facts] *********************************************************
ok: [172.16.130.61]

TASK [webservers : install httpd package] **************************************
changed: [172.16.130.61]

TASK [webservers : install cnf file] *******************************************
changed: [172.16.130.61]

TASK [webservers : start httpd] ************************************************
changed: [172.16.130.61]

TASK [databases : install mysql] ***********************************************
fatal: [172.16.130.61]: FAILED! => {"changed": false, "msg": "No package matching 'mysql-server' found available, installed or updated", "rc": 126, "results": ["No package matching 'mysql-server' found available, installed or updated"]}
...ignoring

TASK [databases : install mariadb] *********************************************
ok: [172.16.130.61]

TASK [databases : install cnf file] ********************************************
changed: [172.16.130.61]

TASK [databases : start mysqld] ************************************************
fatal: [172.16.130.61]: FAILED! => {"changed": false, "msg": "Could not find the requested service mysqld: host"}
...ignoring

TASK [databases : start mariadb] ***********************************************
changed: [172.16.130.61]

RUNNING HANDLER [webservers : restart httpd] ***********************************
changed: [172.16.130.61]

RUNNING HANDLER [databases : restart mysqld] ***********************************
fatal: [172.16.130.61]: FAILED! => {"changed": false, "msg": "Could not find the requested service mysqld: host"}
...ignoring

RUNNING HANDLER [databases : restart mariadb] **********************************
changed: [172.16.130.61]

PLAY RECAP *********************************************************************
172.16.130.100             : ok=6    changed=0    unreachable=0    failed=0   
172.16.130.17              : ok=4    changed=0    unreachable=0    failed=0   
172.16.130.61              : ok=12   changed=7    unreachable=0    failed=0 
```

## 以下内容摘抄自 << 奔跑吧 ansible >>

随 ansible 一起发布的模块都是由 python 编写的，但是实际上模块可以使用任何语言编写

在 yaml 中，字符串是否被引引来，是没有区别的。但是需要注意当有 {{vars}} 开头的字符串，需要引起来，防止歧义

如果希望打印出奶牛，安装`cowsay`<br>
如果不希望打印出奶牛，`export ANSIBLE_NOCOWS=1`，或者修改`ansible.cfg`，在`[defaults]`下添加`nocows=1`

---

在 yaml 中，可以使用`>`来标记折行，yaml 解释器会将换行符替换为空格

```yml
a: >
  Beijing,
  China

b: Beijing, China
```

- sudo
  - 如果为真，ansible 在运行 task 的时候，会切换为 root，
  - 比如 Ubuntu 默认不允许 root 远程登录

---

- yaml `真`：true True TRUE yes Yes YES Y y ON on On
- yaml `否`：false False FALSE no NO No off Off n N


一个合法的 json 也一定是合法的 yaml，因为 yaml 允许字符串被引起来，也将 true false 设为合理的布尔值，并且还支持与 json 数组和对象相同的内联列表与内联字典。

除此之外，yaml 比 json 有更好的阅读性

---

playbook 中，`- name` 是可选的，但是建议加上，作为提示和注解。还可以作为程序启动的标志：使用命令 `--start-at-task taskName` 告诉程序，从中间的 task 开始运行。

---

有一点非常重要:从Ansible前端所使用的YAML解释器角度来看，参数将被按照字符串处理，而不是字典。这意味着如果你想将参数分割为多行的话，你需要像如下这样使用折行语法：

```yml
- name: install nginx
  apt: >
    name=nginx
    update_cache=yes
    # 在安装软件之前，更新软件包缓存 apt-get update
```

Ansible还支持一个旧版本语法:它使用action作为key,将模块的名字也放到value中。<br>
按照这个语法，前面的范例可以写成这样:

```yml
- name: install nginx
  action: apt name=nginx update_ cache=yes
```

---

添加 TLS 支持
pass

---

## 什么时候必须使用引号

如果你刚好在模块声明之后引用变量，YAML解析器会将这个变量引用误解为内联字典。思考一下这个范例的运行结果:

```yml
- name: perform some task
  command: {{ myapp }} -a foo
```

Ansible会尝试将`{{ myapp }} -a foo`的第一部分解析为字典而不是字符串，然后返回一个错误。这种情况下，你必须使用引号将参数引起来:

```yml
- name: perform some task
  command: "{{ myapp }} -a foo"
```

在你的参数中含有冒号的时候也会产生类似的问题。例如:

```yml
- name: show a debug message
  debug: msg="The debug module will print a message: neat, eh?"
```

msg参数中的冒号会误导YAML解析器。你需要使用引号将整个参数字符串引起来，以避免这个问题。

不幸的是，仅仅将参数字符串引起来并不能解决这个问题。

```yml
- name: show a debug message
  debug: "msg=The debug module will print a message: neat, eh?"
```

嗯，这下YAML解析器开心了，但是输出可不是你想要的

```yml
TASK: [show a debug message] **********************************
ok; [localhost] => {
"msg": "The"
}
```

debug模块的msg参数需要将字符串使用引号引起来以捕获空格。在这个特定的范例中，整个参数字符串和msg参数都需要使用引号引起来。好在Ansible同时支持单引号和双引号，所以你可以这样做:

```yml
- name: show a debug message
  debug: "msg='The debug module will print a message: neat, eh?'"
```

这样就会得到我们期望的输出了:

```yml
TASK: [show a debug message ]
************************************
ok: [localhost] => {
"msg": "The debug module will print a message: neat, eh?"
}
```

如果你忘记在正确的地方使用引号，最终以一个非法的YAML文件执行，Ansible 会输出友好易读的错误信息。

---











