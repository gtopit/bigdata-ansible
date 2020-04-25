### 本文档适用自动化部署oracle JDK1.8 + apache-zookeeper-3.4 hadoop2.10 + hbase1.4 + apache-phoenix-4.15.0-HBase-1.4
#### *特别注意：hadoop为单master，hbase压缩包的lib下已包含phoenix-4.15.0-HBase-1.4-server.jar，因此请自行将其包含*  
---
### 开始之前  
安装包的目录和安装目录需要分开
如需要配置免密登录，首先确保被控主机存在ssh所需的公钥和私钥，并且所有主机之间已经开启允许ssh远程登录。
如果不满足以上条件，则可以参考下面操作：
1、开启允许ssh远程登录（所有主机）
打开/etc/ssh/sshd_config，找到PasswordAuthentication，将其设置为yes，执行systemctl restart sshd命令重启ssh服务。
2、配置公钥和私钥（被控主机）
使用ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa命令生成公钥和私钥
如需要安装jdk，请提供jdk压缩包
首先更新下yum源，yum install -y epel-release，然后安装ansible，yum install -y ansible
打开/etc/ansible/ansible.cfg，将host_key_checking = False放开（去掉注释）。

---
#### 使用示例1  
自动化一键部署jdk、zookeeper、Hadoop、hbase  
1、涉及到的服务器及软件安装清单如下：
主机|主机名|部署软件
:---:|:---:|:---:
192.168.233.4|zk01、node1|zookeeper、Hadoop(namenode、secondarynamenode)、hbase(master)
192.168.233.5|zk02、node2|zookeeper、Hadoop(datanode)、hbase(backup master、regionserver)
192.168.233.6|zk03、node3|zookeeper、Hadoop(datanode)、hbase(regionserver)  

---
如果想要自定义软件安装的服务器，请修改hosts文件；如果想要修改其他配置（如软件压缩包所在目录，安装目录），请修改group_vars/all.yml文件。  
默认需要安装的软件放在/usr/local/software，安装目录在/usr/local/installed。  
使用步骤：  
1、在解压后的顶级目录（即hadoop-ansible目录）执行*ansible para -i hosts -m authorized_key -a "user=root key='{{ lookup('file','~/.ssh/id_rsa.pub') }}'"* 命令，实现免密登录。  
2、继续执行*ansible-playbook -i hosts deply-all.yml* 命令，等待其执行完成，即安装成功。  
3、如果执行过程中出现错误，可以不用再执行已成功部署的部分，可以使用如*ansible-playbook -i hosts -t namenode deply-all.yml* 的命令指定需要安装的步骤。所有的步骤信息在deply-all.yml 的tasks.tags 属性下。  
