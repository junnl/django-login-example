# 说明
使用2个django项目，来模拟前后端分离架构。
## demo_login
前端登录页面，采用bootstrap开发。使用ajax发送请求给api

## demo_api
后端api接口，采用rest framework实现，版本为3.11.0。

使用cors组件解决跨域问题

# 1.0
## 环境说明
```bash
1台服务器
操作系统：centos 7.6
配置：2核4g
软件：安装pyton3.5.2以及nginx1.16.1
ip：192.168.31.229
```

直接使用python3运行项目即可
## 前端
```bash
cd django-login-example/1.0/demo_login
pip3 install -r requirements.txt
python3 manage.py runserver 0.0.0.0:8000
```
## api

```bash
cd django-login-example/1.0/demo_api
pip3 install -r requirements.txt
python3 manage.py runserver 0.0.0.0:8001
```

## 配置nginx
将nginx_conf放入`/etc/nginx/conf.d`目录，并启动nginx
```bash
yum install -y nginx
cp django-login-example/1.0/nginx_conf/* /etc/nginx/conf.d
# 启动nginx
nginx
```

## 配置域名解析
如果没有dns，请修改windows 10的hosts文件，添加2条记录
```bash
192.168.31.229 h5.baidu.com
192.168.31.229 api.baidu.com
```

## 访问页面
```bash
http://h5.baidu.com
```
登录信息
```bash
用户名：xiao
密码：1234
```


# 2.0
## 环境说明
```bash
1台服务器
操作系统：centos 7.6
配置：2核4g
软件：安装docker 19.03.5以及nginx1.16.1
```

使用docker运行
## django基础镜像
```bash
cd django-login-example/2.0/django_dockerfile
docker build -t django:2.2.4 .
```

## 前端
这里面的demo_login.tar.gz是1.0中django项目进行了压缩打包。
```bash
cd django-login-example/2.0/login_dockerfile
docker build -t demo_login:v1 .
docker run -it -p 8000:8000 demo_login:v1
```

## api
这里面的demo_api.tar.gz是1.0中django项目进行了压缩打包。
```bash
cd django-login-example/2.0/api_dockerfile
docker build -t demo_api:v1 .
docker run -it -p 8001:8001 demo_api:v1
```
## 配置nginx
无需改动，使用1.0配置即可。

## 访问页面
同上

# 3.0
## 环境说明
```bash
4台服务器，1台master，2台node节点，1台harbor
k8s集群如下：
操作系统：centos 7.6
配置：2核4g
软件：安装Kubernetes 1.16.3，docker 19.03.5。

harbor信息如下：
操作系统：centos 7.6
配置：2核4g
软件：docker 19.03.5，docker-compose 1.24.1，harbor 1.8.0
ip：192.168.31.37
```

## 推送镜像
```bash
docker login 192.168.31.37 -u admin -p Harbor12345
docker tag demo_api:v1 192.168.31.37/library/demo_api:v1
docker push 192.168.31.37/library/demo_api:v1

docker tag demo_login:v1 192.168.31.37/library/demo_login:v1
docker push 192.168.31.37/library/demo_login:v1
```

## k8s发布应用
## 安装ingress
```bash
kubectl apply -f https://kuboard.cn/install-script/v1.17.x/nginx-ingress.yaml
```

## 前端
这里使用2.0中的镜像，请确保将此镜像推送到harbor中。
```bash
cd django-login-example/3.0
kubectl apply -f login.yaml
kubectl apply -f login-ingress.yaml 
```
`注意：请修改yaml文件中的仓库地址，改为实际环境中的harbor`

## api
这里使用2.0中的镜像，请确保将此镜像推送到harbor中。
```bash
cd django-login-example/3.0
kubectl apply -f api.yaml
kubectl apply -f api-ingress.yaml 
```
`注意：请修改yaml文件中的仓库地址，改为实际环境中的harbor`

## 查看pods
```bash
# kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP              NODE         NOMINATED NODE   READINESS GATES
svc-api-6559bb7cb8-97lwp     1/1     Running   0          3h21m   10.244.85.202   k8s-node01   <none>           <none>
svc-login-66c8d579b5-5g46d   1/1     Running   0          3h32m   10.244.85.201   k8s-node01   <none>           <none>
```
确保处于`Running`状态

## 查看ingress
```bash
# kubectl get ingresses.extensions
NAME        HOSTS           ADDRESS   PORTS   AGE
svc-api     api.baidu.com             80      3h23m
svc-login   h5.baidu.com              80      3h37m
```

## 访问页面
在1.0和2.0中，都使用了nginx转发。在3.0就无需nginx转发了，直接使用ingress转发。

将域名解析到任意node节点的ip即可。

```bash
http://h5.baidu.com
```
登录信息
```bash
用户名：xiao
密码：1234
```

# 详细操作
关于1.0和2.0，请查看链接：

https://www.cnblogs.com/xiao987334176/p/12195722.html
<br/>
<br/>
3.0请参考链接：

https://www.cnblogs.com/xiao987334176/p/12196195.html


<br/>
<br/>
Copyright (c) 2020-present, xiao You