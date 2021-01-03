# Vue部署到云服务器中

## Vue项目打包

在Vue项目完成后，要将其部署到云服务器中去，我使用的是腾讯云服务器。  
Vue项目中执行`npm run build`命令会在目录中生成`dist`文件夹，其中包含的就是Vue打包好的项目。  

## 服务器配置  
项目部署需要`Nginx`和`Node.js`的支持，使用ssh终端连接服务器，执行以下命令进行安装
```linux
sudo apt install nodejs npm 
sudo apt install nginx
```
在Nginx安装成功后使用浏览器访问服务器地址或域名会显示Nginx的欢迎页面。  
![Nginx欢迎页面](https://ftp.bmp.ovh/imgs/2021/01/8275f49633d7129a.jpg)  
在ssh终端中执行
```linux
cd /var/www/html
ls
```
在该文件夹下可以看到一个`index.nginx-debian.html`文件，打开该文件可以发现这便是nginx的欢迎页面的来源。

## 项目打包文件上传
使用Ftp工具将之前打包好的`dist`文件夹上传到服务器中  
为了方便，我这里直接将其上传到nginx欢迎文件所在目录
```linux
cd /var/www/html
sudo mkdir vue
```
也可以使用`Github`来代替上传  
将项目上传到Github中，再在服务器中使用`git clone`将线上项目克隆到本地
```text
sudo git clone https://github.com/xxx
```
由于clone下来的是源代码，所以需要对其进行打包
```text
sudo npm install    // 安装依赖
sudo npm run build  // 打包
```
这样会在项目文件中生成`dist`文件夹，最终效果与上一个方法一致。

## 配置Nginx
接下去需要修改Nginx配置  
```linux
cd /etx/nginx/sites-available
```
在该文件夹下有一个`default`文件，对其进行修改
```linux
sudo vim default
```
其中一行是：`root/var/www/html`  
这就是为什么我们浏览器访问服务器看到的是Nginx的欢迎页面，我们只要将其改为我们自己Vue项目的dist地址即可
```linux
修改
root /var/www/html
为：
rrot var/www/html/vue/dist
```
保存推出，使用`service nginx restart`重启Nginx来使其生效  
  
接下去再访问ip地址就可以看到我们自己Vue项目的页面。
  
但还有一个问题，在访问这个IP下的任意一个路径时，`例如：ip/abc`，会返回一个404页面，这时需要在nginx的default文件中添加如下设置：
```linux
server_name _ ;
// 在server_name _; 后面添加这一段代码
error_page 404 /;
```
保存重启Nginx，再去访问不管在哪个路径下，看到的都会是Vue的项目页面。