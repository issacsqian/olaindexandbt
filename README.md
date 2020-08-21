# **宝塔面板搭建[OLAINDEX](https://wangningkai.github.io/OLAINDEX/)**

------

本文根据官方文档写成，可能有部分表述不准确，还请指正

测试环境：Google Cloud Platform 香港节点 系统：CentOS 8.1 X64

------



> ## 第一步，安装宝塔面板

**Centos安装命令：**

```
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

**Ubuntu/Deepin安装命令（谁会用deepin做服务器啊）：**

php artisan cache:clear php artisan config:clear

**Debian安装命令：**

```
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh
```

**Fedora安装命令:**

```
wget -O install.sh http://download.bt.cn/install/install_6.0.sh && bash install.sh
```

[**面板管理常用命令：**]: https://www.bt.cn/btcode.html

具体安装教程可以左拐到宝塔官网bt.cn

> ## 第二步，PHP环境搭建

官方文档的要求：

- **PHP 扩展要求**

  - PHP >= 7.2

  - PHP OpenSSL 扩展

  - PHP PDO 扩展

  - PHP Mbstring 扩展

  - PHP Tokenizer 扩展

  - PHP XML 扩展

  - PHP Ctype 扩展

  - PHP JSON 扩展

  - PHP BCMath 扩展

  - PHP Fileinfo 扩展 *

    

那么我们需要做的有这些：

1. 安装LNMP套件 这个在你安装完宝塔以后，第一次进入就会推荐安装LNMP or LAMP。你需要做的就是选择LNMP。

2. 配置PHP模块

   由于我们的要求是PHP>=7.2，这里直接安装PHP7.4 ![php74install](https://s1.ax1x.com/2020/07/18/U2uuqS.png)

   安装的要求，需要我们安装和开启某些模块。

   进入PHP7.4的设置

   选择“安装扩展” 并安装***“fileinfo”和“opcache”***扩展![安装扩展](https://s1.ax1x.com/2020/07/18/U2uTzt.png)

   选择禁用函数

   并把***“proc_open” "proc_get_status" "putenv" "exec" "shell_exec"*** 从列表删除

   ![禁用函数](https://s1.ax1x.com/2020/07/18/U2K8Te.png)

   3.配置composer（命令来自官方文档）

   ```
   curl -sS https://getcomposer.org/installer | php  
   mv composer.phar /usr/local/bin/composer 
   composer config -g repo.packagist composer https://packagist.laravel-china.org # 更换源为国内源，国外服务器可忽略此步骤
   ```

  
   

> ## 第三步，网页配置

在这一步我们需要对网页进行配置，包括伪静态，mysql配置修改等等。

1. 新建一个网页，配置保持默认。  ![新建网页](https://s1.ax1x.com/2020/07/18/U2Ml3n.png)

2. 配置伪静态，规则为laravel5 ![伪静态](https://s1.ax1x.com/2020/07/18/U2MyDK.png)



3. 配置ssl 使用Let's Encrypt即可


4. 随后需要用到终端。命令来自于官方文档。

   ### **注意：**

   这里的命令需要一步一步执行

   ```
   cd web目录 #网页配置时设置的目录
   git clone https://github.com/WangNingkai/OLAINDEX.git tmp 
   mv tmp/.git . 
   rm -rf tmp 
   git reset --hard 
   composer install -vvv # 这里确保已成功安装 composer ，如果报权限问题，建议给予用户完整权限。
   chmod -R 777 storage 
   chown -R www:www * # 此处 www 根据服务器具体用户组而定
   composer run install-app
   ```

​      不出意外就可以看见这样的提示 ![success](https://s1.ax1x.com/2020/07/18/U2leYQ.png)

如果出现错误可以尝试
一，清除缓存和配置

 ```
 php artisan cache:clear 
 php artisan config:clear
 ```
二，换用5.0分支 具体是在git clone时使用如下命令

```
git clone -b 5.0 https://github.com/WangNingkai/OLAINDEX.git tmp 
```

二，修改mysql配置
定位到服务器的mysql文件夹 一般情况在 **/www/server/mysql**
   找到**my.cnf** 对它编辑 ![my.cnf](https://s1.ax1x.com/2020/07/18/U2QPVU.png)   在[mysqld]一行之下添加  `skip-grant-tables`       
    

![skip](https://s1.ax1x.com/2020/07/18/U2Q3PH.png)
       然后保存。
    


5.设置运行目录为/public ![运行目录](https://s1.ax1x.com/2020/07/18/U2QIJJ.png)

至此已经可以正常访问了

> ## 第四步，导入账号

鉴于微软不知道出于什么原因关闭了原本的应用注册通道，我们需要在微软钦定的新渠道，即Azure手动应用注册。

在网页配置完成之后 浏览器打开https://[你的域名]/admin

然后在后台新增账号

默认登陆用户名是admin 密码是123456![](https://s1.ax1x.com/2020/08/19/d1NSLq.png)

点击新增账号 可以看到需要我们填写的有回调地址，client_id，client_secret

其中回调地址 redirect_uri 为https://[你的域名]/callback

其余两项需要申请。

1. 打开https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview 并在左侧栏找到应用注册 进入。![](https://s1.ax1x.com/2020/08/19/d1NY6A.png)

   名称可以随便填 受支持的账户类型选择第三个 重定向URL填写

   ```
   https://你的域名/callback
   ```

   

2. client_id即应用程序(客户端)ID 记录下来![](https://s1.ax1x.com/2020/08/19/d1NJld.png)

3. 左边栏选择证书和密码  点击新客户端密码 生成密码 即client_secret![](https://s1.ax1x.com/2020/08/19/d1N3fe.png)

4. 授予权限  看图![](https://s1.ax1x.com/2020/08/19/d1NGSH.png)

   ![](https://s1.ax1x.com/2020/08/19/d1N1YD.png)

   ![](https://s1.ax1x.com/2020/08/19/d1NlFO.png)

5.回填参数![](https://s1.ax1x.com/2020/08/19/d10IyQ.png)



![](https://s1.ax1x.com/2020/08/19/d1BFFx.png)

此时网页已经可以正常访问了！

![](https://s1.ax1x.com/2020/08/19/d1B3ff.png)
