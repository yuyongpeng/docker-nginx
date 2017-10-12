### 这个是基于dockerhub的官方nginx的修改版本

生成了一个debug的版本

* nginx安装路径：/usr/local/nginx
* 配置文件路径：/usr/local/nginx/conf
* modules路径：/usr/local/nginx/moules

### 运行方式

* 先安装docker
* git clone https://github.com/yuyongpeng/docker-nginx.git
* cd docker-nginx/alpine 
* docker build -t nginx:alpine .
* docker run -d nginx:alipine
* docker exec -ti xxxxxxxx bash