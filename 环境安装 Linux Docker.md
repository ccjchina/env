# 环境安装

## 基本命令 

~~~sh
 
#移动
mv mycat/  /usr/server/
#copy
cp xxx  /xxx
#删除
rm /f xxx
#解压
tar -zxvf xxx.tar.gz

#查看防火墙状态命令
systemctl status firewalld.service
#关闭
systemctl stop firewalld.service #关闭运行的防火墙
firewall-cmd --zone=public --add-port=2375/tcp --permanent
firewall-cmd --reload
#查询端口是否开放
firewall-cmd --query-port=8080/tcp
#开放80端口
firewall-cmd --permanent --add-port=80/tcp
#移除端口
firewall-cmd --permanent --remove-port=8080/tcp
#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
#查看端口使用
netstat -an | grep 2375
~~~

## 任务计划

~~~shell
####计划任务
yum install crontabs
#（设为开机启动）
systemctl enable crond 
#（启动crond服务）
systemctl start crond
#（查看状态）
systemctl status crond
#配置
vi /etc/crontab
~~~

## 部署微服务

~~~shell
##部署服务，但是服务器重启后，不会自动启动。

#查找
ps -aux|grep EkuaibaoAuthSvc-1.0.0.jar
#按端口 查找
lsof -i:10089
#关闭
kill -9 28038

#copy to 
# sudo chmod 754 [folderName]  
##default /tmp has permission
scp EkuaibaoAuthSvc-1.0.0.jar root@192.168.1.30:/usr/app

#后台启动
nohup  java -jar /usr/app/EkuaibaoAuthSvc-1.0.0.jar &
~~~

## nginx install

~~~sh
#nginx
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

sudo yum install -y nginx
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
curl localhost  #default port 80

#禁止 yum 自动更新
vim /etc/yum.conf 
exclude=nginx
~~~

## docker install

~~~sh
#docker 安装
yum update
yum -y remove docker docker-common docker-selinux docker-engine
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

##启动并加入开机启动
systemctl start docker
#重启命令   
systemctl restart docker
#开机启动
systemctl enable docker
#查看docker版本号
docker version

systemctl status docker #查看docker运行状态
#Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)

#私有仓库
docker pull registry
docker run -d -p 5000:5000 --restart=always --name registry registry
#证书 https://www.jianshu.com/p/b93feaf43f37
#https://<ip>:<port>/v2/_catalog
curl -i 127.0.0.1:5000/v2/
curl http://127.0.0.1:5000/v2/_catalog
docker logs -f -t --since="2021-01-15" --tail=500 registry

# vim /usr/lib/systemd/system/docker.service
#ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock


#[root@open-api ~]# docker pull 192.168.1.30/app:1.0.0
#Error response from daemon: Get https://192.168.1.30/v2/: dial tcp 192.168.1.30:443: connect: connection refused

sudo systemctl daemon-reload
sudo systemctl restart docker.service

#TEST 
docker -H tcp://ip:2375 version
docker -H tcp://ip:2375 images

#清理
docker image prune -f

####Docker pull 提示manifest unknown的报错

#window 环境变量  tcp://192.168.1.30:2375
set DOCKER_HOST=tcp://localhost:2375

#Linux 环境变量
export DOCKER_HOST=tcp://localhost:2375

#maven 打包  https://www.cnblogs.com/jpfss/p/10945324.html
set DOCKER_HOST=tcp://localhost:2375 && mvn clean package dockerfile:build
set DOCKER_HOST=tcp://192.168.1.30:2375 && mvn clean package dockerfile:build

#上传到私有仓库
mvn dockerfile:push

#sample docker run   部署
docker run --name goodwe-hello -d -p 8080:8080 192.168.1.30:5000/app:1.0.0
#查看运行的容器
docker ps -a
#使得容器随，docker重启时，自己启动。
docker update --restart=always 1e51ee541820

#Docker Compose
## TODO:
https://blog.csdn.net/guizaian/article/details/108179742

#查询镜像
curl  <仓库地址>/v2/_catalog

#查询镜像tag(版本)
curl  <仓库地址>/v2/<镜像名>/tags/list

#删除镜像API
curl -I -X DELETE "<仓库地址>/v2/<镜像名>/manifests/<镜像digest_hash>"

#获取镜像digest_hash
curl  <仓库地址>/v2/<镜像名>/manifests/<tag> \
	--header "Accept: application/vnd.docker.distribution.manifest.v2+json"

~~~

## docker 阿里云镜像加速

~~~json
{
  "registry-mirrors": [
    "https://yfzcel92.mirror.aliyuncs.com"
  ],
  "insecure-registries": [
    "172.0.0.1:5000",
    "192.168.1.30:5000"
  ]
}
~~~



## K8S

~~~sh
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
apt-get update
apt-get install -y kubelet kubeadm kubectl
~~~



## sentinel  dashboard

~~~sh
#sentinel dashboard
docker pull bladex/sentinel-dashboard
docker run --name sentinel -d -p 8858:8858 -d bladex/sentinel-dashboard
~~~



~~~shell
#build springboot docker image

~~~



~~~tex
I/O exception (java.io.IOException) caught when processing request to {}->unix://localhost:80: Broken pipe

导致这个错误的原因是 project.artifactId 可能包含了大写。见pom.xml

https://blog.csdn.net/CoreyXuu/article/details/109336018

1.dockerfile 读取的到的环境出现问题。如dockerfile 的 add 文件是springdemo.jar ,结果打包成了spring.test.jar

2.pom配置出现问题会导致读取配置出香异常

3.maven 的配置setting.xml，需要配置group

4.关于docker 执行命令的时候是否是更目录也可能出现问题

5.打包命令 是dockerfile 还是 Dockerfile
~~~



## keytool genkey

~~~cmd
E:\>keytool -genkey -alias goodweapi -keyalg RSA -keypass mssdtg2 -keystore gwdapi.jks -storepass mssdtg2
您的名字与姓氏是什么?
  [Unknown]:  gwd
您的组织单位名称是什么?
  [Unknown]:  com
您的组织名称是什么?
  [Unknown]:  gwd
您所在的城市或区域名称是什么?
  [Unknown]:  sz
您所在的省/市/自治区名称是什么?
  [Unknown]:  sz
该单位的双字母国家/地区代码是什么?
  [Unknown]:  chs
CN=gwd, OU=com, O=gwd, L=sz, ST=sz, C=chs是否正确?
  [否]:  是

Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore gwdapi.jks -destkeystore gwdapi.jks -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。
~~~

## mysql install

~~~sql
#MYSQL

mysqld --install MySQL_3306 --defaults-file="D:\env\mysql-8.0.25-winx64\my.ini"

mysqld --initialize --console

ALTER USER 'root'@'localhost' IDENTIFIED BY 'password_new';

mysql> use mysql;
Database changed

--不修改此处会有问题
mysql> update user set host='%' where user='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
### WAIT A MOUNT..
mysql> GRANT ALL ON *.* TO 'root'@'%';
Query OK, 0 rows affected (0.01 sec)

FLUSH PRIVILEGES;


#Linux
wget http://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
rpm -ivh mysql80-community-release-el7-3.noarch.rpm


select version() from dual;   ##版本
service mysqld restart  ##重启


##MYSQL_HOME


## 8.0 
mysql -uroot -p123456 -h127.0.0.1 -P8066 --default_auth=mysql_native_password
INSERT INTO users(name,indate) VALUE('1',date());
INSERT INTO users(name,indate) VALUE('2',date());
INSERT INTO users(name,indate) VALUE('3',date());
~~~



## rabbitmq

~~~shell
#pull
docker pull rabbitmq:latest
#run
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:latest

#rabbitmq cmd shell
##管理后台
rabbitmq-plugins enable rabbitmq_management

##用户管理
rabbitmqctl  add_user  user_name pass_word
rabbitmqctl  set_permissions  user_name  ".*"  ".*"  ".*"
rabbitmqctl  set_user_tags user_name administrator

##修改密码
rabbitmqctl  change_password  Username  'xxx'
~~~





## redis

~~~cmd
#Redis

#LOGIN CMD
.\redis-cli -p 6379 -a 123456
redis-cli -h 192.168.1.30 -p 6379 -a ^adm4rds$


#Linux 重启 rds
sudo systemctl restart redis
#开机启动
systemctl enable redis



~~~

## node js

~~~sh
#node
npm config set registry https://registry.npm.taobao.org 
npm info underscore
npm install -g yarn
yarn --version
# yarn config set registry https://registry.npm.taobao.org
~~~



~~~bash
#保存文件时用  : tee 用于读取输入文件，同时保存; %表示当前编辑文件;
w ! sudo tee %


~~~





