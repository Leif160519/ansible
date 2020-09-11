## 一、说明
本项目源自：[centos-script](https://github.com/Leif160519/centos-script) | [ubuntu-script](https://github.com/Leif160519/ubuntu-script)

针对待执行脚本的主机数量众多的情况下，建议采用ansible刷入的方式统一配置这些机器，可以大大减少人工成本与时间成本

项目目录结构
```
ansible-linux
├── files
│   ├── docker
│   │   ├── daemon.json
│   │   └── docker-ce.repo
│   ├── network
│   │   ├── 00-installer-config.yaml
│   │   ├── ifcfg-eth0
│   │   ├── interfaces
│   │   └── resolv.conf
│   └── repository
│       ├── CentOS-Base.repo
│       ├── sources-debian.list
│       └── sources.list
├── inventory
│   ├── all
│   └── dist
├── playbooks
│   ├── config.yml
│   ├── docker.yml
│   ├── files -> ../files
│   ├── gitlab.yml
│   ├── install.yml
│   ├── network.yml
│   ├── ntp.yml
│   ├── oh-my-fish.yml
│   ├── oh-my-zsh.yml
│   ├── rar.yml
│   ├── remove_gnome.yml
│   ├── repository.yml
│   ├── ssh-key.yml
│   └── universal.yml
└── README.md
```

## 二、如何使用

- 1.选择一台Linux或Mac主机安装python3并安装ansible
```
#ubuntu
apt-get install ansible

#centos
yum -y install ansible

#macos
brew install ansible #没有装brew包管理器的自行百度安装一下
```

- 2.将本机的ssh-key全部复制进所有主机中
```
ssh-copy-id -i root@<host-ip>
```

- 3.克隆本项目
```
git clone https://github.com/Leif160519/ansible-linux
```

- 4.修改inventory下的主机hostname和IP地址
```
# all
[all]
ubuntu-2004       ansible_host=192.168.0.108
centos-7          ansible_host=192.168.0.109
debian-10         ansible_host=192.168.0.110

#dist
[dist]
[dist:children]   
dist.debian
dist.redhat

[dist.debian:children]         # debian子类
dist.ubuntu
dist.debian9
dist.debian10

[dist.redhat:children]         # redhat子类
dist.centos

[dist.ubuntu:children]         # ubuntu子类
dist.ubuntu.lts

[dist.centos:children]         # centos子类
dist.centos6
dist.centos7

[dist.ubuntu.lts:children]     # ubuntu发行版子类
dist.u1604
dist.u1804
dist.u2004

[dist.debian9]                 # debian9机器列表

[dist.debian10]                # debian10机器列表
debian-10

[dist.centos6]                 # centos6机器列表

[dist.centos7]                 # centos7机器列表
centos-7

[dist.u1604]                   # ubuntu16.04机器列表

[dist.u1804]                   # ubuntu18.04机器列表

[dist.u2004]                   # ubuntu20.04机器列表
ubuntu-2004
```

- 5.ansible配置文件配置忽略组语法错误
```
vi /etc/ansible/ansible.cfg

[defaults]
force_valid_group_names = ignore
···

```
- 6.开始刷入
```
ansible-playbook -u root -i inventory/ playbooks/universal.yml
```

## 三、特定情况
- 1.安装某些软件
```
ansible-playbook -u root -i inventory/ playbooks/install.yml -t [ubuntu | install] 
```

- 2.在某些机器上指定playbook
```
ansible-playbook -u root -i inventory/ playbooks/install.yml -l 'ubuntu-2004 centos-7 debian-10'
```

- 3.监控主机安装prometheus
执行`prometheus.yml`之后别忘了执行一下`node_exporter.yml`，否则`9090`不能显示监控数据

- 4.被监控主机安装node_exporter
需在`node_export.yml`文件中将`delegate_to`后面的`prometheus`主机名按照实际情况修改之后再执行

- 5.安装node_exporter时报错解决办法
在某些情况下，执行带有`delegate_to`（任务委派）的playbook时候，会报错：
```
failed: [centos-7 -> 192.168.0.108] (item=  - job_name: 'node') => {"ansible_loop_var": "item", "changed": false, "item": "  - job_name: 'node'", "module_stderr": "Shared connection to 192.168.0.108 closed.\r\n", "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n", "msg": "The module failed to execute correctly, you probably need to set the interpreter.\nSee stdout/stderr for the exact error", "rc": 127}

failed: [debian-10 -> 192.168.0.108] (item=  - job_name: 'node') => {"ansible_loop_var": "item", "changed": false, "item": "  - job_name: 'node'", "module_stderr": "Shared connection to 192.168.0.108 closed.\r\n", "module_stdout": "/bin/sh: 1: /usr/bin/python3.7: not found\r\n", "msg": "The module failed to execute correctly, you probably need to set the interpreter.\nSee stdout/stderr for the exact error", "rc": 127}
```

原因是ansible未检测到python环境，也就是说你在Linux终端打`python`是提示无此命令的，所以解决办法是，强制ansible使用python3环境，在执行命令后面添加`-e ansible_python_interpreter=/usr/bin/python3`参数即可：
```
ansible-playbook -i inventory playbooks/node_exporter.yml  -l 'debian-10' -e ansible_python_interpreter=/usr/bin/python3
```

## 四、写在最后
由于个人精力有限，项目并不会经常更新，若大家有兴趣，可以在项目中贡献自己的代码，一起将[ansible-linux](https://github.com/Leif160519/ansible-linux)发展壮大，感谢各位的理解与支持！
