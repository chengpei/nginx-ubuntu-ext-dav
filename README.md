# 需求来源
起初想在家里软路由上的nginx里配置webdav相关支持实现轻NAS功能，结果发现我使用的官方nginx镜像并没有安装nginx-dav-ext-module模块，于是找到了nginx官方的提供的dockerfile文件，准备进行修改追加上该模块。
我使用的nginx版本1.26.2对应的Dockerfile： [https://github.com/nginxinc/docker-nginx/blob/e78cf70ce7b73a0c9ea734c9cf8aaaa283c1cc5a/stable/debian/Dockerfile](https://github.com/nginxinc/docker-nginx/blob/e78cf70ce7b73a0c9ea734c9cf8aaaa283c1cc5a/stable/debian/Dockerfile)
# 遇到的问题
 官方构建nginx镜像使用的是apt安装，并不是源码编译，所以我这里没法再编译时增加模块，所以我想到直接用apt命令安装dav扩展的模块
```
apt install -y nginx-extras
```
我在ubuntu虚拟机上这么安装nginx以及相关扩展包没有问题，但是在官方的Dockerfile镜像中增加该命令结果构建报错了，原因是版本冲突了
# 解决方案
最后决定干脆不修改官方的镜像的，基于ubuntu镜像、nginx源码以及nginx-dav-ext-module模块定制一个景象，之所以选择ubuntu是因为我在虚拟机上编译成功了，所以应该不会有什么问题，无非就是最终构建出来的镜像比较大，自己用也无所谓了。
# 源码
其中nginx-1.26.2.tar.gz 是下载的nginx源码，然后在其中增加了nginx-dav-ext-module模块的源码压缩放一起的，你也可以自己下载需要的版本进行替换
nginx源码：[https://github.com/nginx/nginx](https://github.com/nginx/nginx)
dav-ext源码：[https://github.com/arut/nginx-dav-ext-module](https://github.com/arut/nginx-dav-ext-module)
sh脚本是参考nginx官方镜像启动方式增加的脚本

主要就是基于ubuntu镜像，把nginx相关源码丢进去，然后安装编译需要的工具库，编译源码安装。
在源码当前目录执行命令构建：
```
docker build -t nginx:1.26.2-ubuntu-ext .
```
构建成功即可运行一个具有dav-ext支持的nginx镜像
```
docker run --restart=always -p80:80 --name nginx-ext -d nginx:1.26.2-ubuntu-ext
```
