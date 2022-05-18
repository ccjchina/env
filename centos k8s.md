~~~shell

#修改源
yum -y install wget
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

yum clean all
yum makecache

#关闭防火墙
firewall-cmd --state	    #查看防火墙状态
systemctl stop firewalld.service    	#停止firewall
systemctl disable firewalld.service    	#禁止firewall开机启动

#关闭selinux
getenforce	#查看selinux状态
setenforce 0	#临时关闭selinux
sed -i 's/^ *SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config	#永久关闭（需重启系统）

yum update -y

reboot


hostnamectl set-hostname master01
more /etc/hostname


cat >> /etc/hosts << EOF
172.27.34.3    master01
172.27.34.4    master02
172.27.34.5    master03
172.27.34.93   work01 
172.27.34.94   work02
172.27.34.95   work03
EOF


swapoff -a
sed -i.bak '/swap/s/^/#/' /etc/fstab


cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet


# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce-17.09.1.ce-1.el7.centos
# Step 4: 开启Docker服务
sudo service docker start

注意：其他注意事项在下面的注释中
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
# 注意：在某些版本之后，docker-ce安装出现了其他依赖包，如果安装失败的话请关注错误信息。例如 docker-ce 17.03 之后，需要先安装 docker-ce-selinux。
# yum list docker-ce-selinux- --showduplicates | sort -r
# sudo yum -y install docker-ce-selinux-[VERSION]

# 通过经典网络、VPC网络内网安装时，用以下命令替换Step 2中的命令
# 经典网络：
# sudo yum-config-manager --add-repo http://mirrors.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo
# VPC网络：
# sudo yum-config-manager --add-repo http://mirrors.could.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo


docker version


vim /etc/yum.conf
exclude=docker*

~~~

~~~shell
# 只在Master节点上执行即可，主要用于分发文件和远程执行命令
NODE_IPS=("10.199.142.31" "10.199.142.32" "10.199.142.33" "10.199.142.34" "10.199.142.35")
for node_ip in ${NODE_IPS[@]};do
    echo ">>> ${node_ip}"
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@${node_ip}
done
~~~





~~~shell
docker run \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-p 3000:80 \
-d --name vueApp \
vuenginxdemo
~~~

