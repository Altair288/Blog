---
layout: post
title: Linux手动部署LNMP及Wordpress完整教程
date: 2024-01-04
Author: 
categories: 
tags: [Linux]
comments: false
toc: true
---


# 部署LNMP环境（CentOS 7）

## 准备工作

快速部署时选择已有实例或手动部署LNMP环境时，已有实例必须满足以下条件：

- 实例已分配公网IP地址或绑定弹性公网IP（EIP）。
- 操作系统必须为CentOS 7.x。
- 实例安全组的入方向规则已放行22、80、443端口。具体操作，请参见添加安全组规则。

## 操作步骤

### 步骤一：关闭防火墙和SELinux

1. 远程连接需要部署LNMP环境的ECS实例。具体操作，请参见连接方式概述。

2. 关闭防火墙。

    a. 运行以下命令，查看当前防火墙的状态。

    ```shell
    systemctl status firewalld
    ```

    - 如果防火墙的状态参数是`inactive`，则防火墙为关闭状态
    - 如果防火墙的状态参数是`active`，则防火墙为开启状态

    b. 关闭防火墙。

    - 临时关闭防火墙：

    ```shell
    sudo systemctl stop firewalld
    ```

    - 永久关闭防火墙：

    1. 关闭防火墙。

    ```shell
    sudo systemctl stop firewalld
    ```

    2. 实例开机时，禁止启动防火墙服务。

    ```shell
    sudo systemctl disable firewalld
    ```

3. 关闭SELinux。

    a. 运行以下命令，查看SELinux的当前状态。

    ```shell
    gentenforce
    ```

    - 如果SELinux状态参数是Disabled，则SELinux为关闭状态，请执行步骤二：安装Nginx。

    - 如果SELinux状态参数是Enforcing，则SELinux为开启状态，请执行

    ```shell
    setenforce 0
    ```

### 步骤二：安装Nginx

1. 运行以下命令，安装Nginx。

    ```shell
    sudo yum -y install nginx
    ```

2. 运行以下命令，查看Nginx版本。

    ```shell
    nginx -v
    ```

3. 返回结果类似如下所示，表示Nginx安装成功。

    ```shell
    nginx version: nginx/1.20.1
    ```

### 步骤三：安装并配置MySQL

#### 安装MySQL

1. 运行以下命令，更新YUM源。

    ```shell
    sudo rpm -Uvh  http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
    ```

2. 运行以下命令，安装MySQL。

    > __说明__  如果提示报错信息No match for argument，您需要先运行命令sudo yum module disable mysql禁用默认的MySQL模块，再安装   MySQL。

    ```shell
    sudo yum -y install mysql-community-server --nogpgcheck
    ```

3. 运行以下命令，查看MySQL版本号。

    ```shell
    mysql -V
    ```

    返回结果如下所示，表示MySQL安装成功。

    ```shell
    mysql  Ver 14.14 Distrib 5.7.42, for Linux (x86_64) using  EditLine wrapper
    ```

4. 运行以下命令，启动MySQL。

    ```shell
    sudo systemctl start mysqld
    ```

5. 依次运行以下命令，设置开机启动MySQL。

    ```shell
    sudo systemctl enable mysqld
    sudo systemctl daemon-reload
    ```

#### 配置MySQL

1. 运行以下命令，查看`/var/log/mysqld.log`文件，获取并记录`root`用户的初始密码。

    ```shell
    sudo grep 'temporary password' /var/log/mysqld.log
    ```

    命令行返回结果如下，其中`ARQTRy3+****`为MySQL的初始密码。在下一步重置root用户密码时，会使用该初始密码。

    ```shell
    2021-11-10T07:01:26.595215Z 1 [Note] A temporary password is generated for root@localhost: ARQTRy3+****
    ```

2. 运行以下命令，配置MySQL的安全性。

    ```shell
    sudo mysql_secure_installation
    ```

    1. 输入MySQL的初始密码。

    > __说明__ 在输入密码时，系统为了最大限度的保证数据安全，命令行将不做任何回显。您只需要输入正确的密码信息，然后按Enter键即可。

    ```shell
    Securing the MySQL server deployment.
    Enter password for user root: #输入上一步获取的root用户初始密码
    ```

    2. 设置MySQL的新密码。

    ```shell
    The existing password for the user account root has expired. Please set a new password.

    New password: #输入新密码。长度为8至30个字符，必须同时包含大小写英文字母、数字和特殊符号。特殊符号包含()` ~!@#$%^&*-+=| {}[]:;‘<>,.?/

    Re-enter new password: #确认新密码。
    The 'validate_password' plugin is installed on the server.
    The subsequent steps will run with the existing configuration
    of the plugin.
    Using existing password for root.

    Estimated strength of the password: 100 #返回结果包含您设置的密码强度。
    Change the password for root ? (Press y|Y for Yes, any other key for No) :Y #您需要输入Y以确认使用新密码。

    #新密码设置完成后，需要再次验证新密码。
    New password:#再次输入新密码。

    Re-enter new password:#再次确认新密码。

    Estimated strength of the password: 100
    Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) :Y #您需要输入Y，再次  确认使用新密码。
    ```

    3. 输入`Y`删除匿名用户。

    ```shell
    Remove anonymous users? (Press y|Y for Yes, any other key for No) :Y
    Success.
    ```

    4. 输入`Y`禁止使用root用户远程登录MySQL。

    ```shell
    Disallow root login remotely? (Press y|Y for Yes, any other key for No) :Y
    Success.
    ```

    5. 输入`Y`删除test库以及用户对test库的访问权限。

    ```shell
    Remove test database and access to it? (Press y|Y for Yes, any other key for No) :Y
     - Dropping test database...
    Success.

     - Removing privileges on test database...
    Success.
    ```

    6. 输入`Y`重新加载授权表。

    ```shell
    Reload privilege tables now? (Press y|Y for Yes, any other key for No) :Y
    Success.

    All done!
    ```

### 步骤四：安装并配置PHP

#### 安装PHP

1. 更新YUM源。

    1. 运行以下命令，添加EPEL源。

    ```shell
    sudo yum install \
    https://repo.ius.io/ius-release-el7.rpm \
    https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    ```

    2. 运行以下命令，安装PHP。

    ```shell
    sudo yum -y install php70w-devel php70w.x86_64 php70w-cli.x86_64 php70w-common.x86_64 php70w-gd.x86_64 php70w-ldap.x86_64 php70w-mbstring.x86_64 php70w-mcrypt.x86_64  php70w-pdo.x86_64   php70w-mysqlnd  php70w-fpm php70w-opcache php70w-pecl-redis php70w-pecl-mongodb
    ```

    3. 运行以下命令，查看PHP版本。

    ```shell
    php -v
    ```

    返回结果如下所示，表示安装成功。

    ```shell
    PHP 7.0.33 (cli) (built: Dec  6 2018 22:30:44) ( NTS )
    Copyright (c) 1997-2017 The PHP Group
    Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
    with Zend OPcache v7.0.33, Copyright (c) 1999-2017, by Zend Technologies                
    ```

#### 修改Nginx配置文件以支持PHP

1. 运行以下命令，备份Nginx配置文件。

    ```shell
    sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
    ```

2. 修改Nginx配置文件，添加Nginx对PHP的支持。

    > __重要__   若不添加此配置信息，后续您使用浏览器访问PHP页面时，页面将无法显示。

    1. 运行以下命令，打开Nginx配置文件。

    ```shell
    sudo vim /etc/nginx/nginx.conf
    ```

    2. 按 `i` 进入编辑模式。

    3. 在 `server` 大括号内，修改或添加下列配置信息。
    除下面提及的需要添加或修改的配置信息外，其他配置保持默认值即可。

        - 添加或修改location /配置信息。

        ```shell
                location / {
            index index.php index.html index.htm;
        }
        ```

        - 添加或修改location ~ .php$配置信息。

        ```shell
                #添加下列信息，配置Nginx通过fastcgi方式处理您的PHP请求。
        location ~ .php$ {
            root /usr/share/nginx/html;    #将/usr/share/nginx/html替换为您的网站根目录，本文使用/usr/share/nginx/html作为网站根目录。
            fastcgi_pass 127.0.0.1:9000;   #Nginx通过本机的9000端口将PHP请求转发给PHP-FPM进行处理。
            fastcgi_index index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include fastcgi_params;   #Nginx调用fastcgi接口处理PHP请求。
        }
        ```

    添加或修改配置信息后，文件内容如下图所示：

    [![piv4hiq.png](https://s11.ax1x.com/2024/01/04/piv4hiq.png)](https://imgse.com/i/piv4hiq)

    4. 按`Esc`键，输入`:wq`，按`Enter`键关闭并保存配置文件。

3. 运行以下命令，启动Nginx服务。

    ```shell
    sudo systemctl start nginx
    ```

4. 运行以下命令，设置Nginx服务开机自启动。

    ```shell
    sudo systemctl enable nginx
    ```

#### 配置PHP

1. 新建并编辑`phpinfo.php`文件，用于展示PHP信息。

    1. 运行以下命令，新建`phpinfo.php`文件。

    ```shell
    sudo vim <网站根目录>/phpinfo.php
    ```

    <网站根目录>是您在`nginx.conf`配置文件中`location ~ .php$`大括号内，配置的`root`参数值，如下图所示。

    [![piv5Fwd.png](https://s11.ax1x.com/2024/01/04/piv5Fwd.png)](https://imgse.com/i/piv5Fwd)

    本文配置的网站根目录为`/usr/share/nginx/html`，因此需要运行以下命令新建`phpinfo.php`文件：

    ```shell
    sudo vim /usr/share/nginx/html/phpinfo.php
    ```

    2. 按 `i` 进入编辑模式。

    3. 输入下列内容，函数phpinfo()​会展示PHP的所有配置信息。

    ```shell
    <?php echo phpinfo(); ?>
    ```

2. 运行以下命令，启动PHP-FPM。

    ```shell
    sudo systemctl start php-fpm
    ```

3. 运行以下命令，设置PHP-FPM开机自启动。

    ```shell
    sudo systemctl enable php-fpm
    ```

### 步骤五：测试访问LNMP配置信息页面

1. 在本地Windows主机或其他具有公网访问能力的Windows主机中，打开浏览器。

2. 在浏览器的地址栏输入`http://<公网IP地址>/phpinfo.php`进行访问。访问结果如下图所示，表示LNMP环境部署成功。

[![piv5NlT.jpg](https://s11.ax1x.com/2024/01/04/piv5NlT.jpg)](https://imgse.com/i/piv5NlT)

### 后续步骤

测试访问LNMP配置信息页面后，建议您运行以下命令将phpinfo.php文件删除，消除数据泄露风险。

```shell
sudo rm -rf <网站根目录>/phpinfo.php
```

其中，<网站根目录>需要替换为您在`nginx.conf`中配置的网站根目录。

本文配置的网站根目录为`/usr/share/nginx/html`，因此需要运行以下命令：

```shell
sudo rm -rf /usr/share/nginx/html/phpinfo.php
```

# 在CentOS 7.x系统的ECS实例上搭建WordPress网站

## 搭建WordPress网站

1. 配置WordPress数据库。

    1. 进入MySQL数据库。
    使用`root`用户登录MySQL，并输入密码。密码为您在搭建环境时为数据库设置的密码。

    ```shell
    mysql -uroot -p
    ```

    2. 为WordPress网站创建数据库。
    本教程中数据库名为`wordpress`。

    ```shell
    create database wordpress;
    ```

    3. 创建一个新用户管理WordPress库，提高安全性。
    MySQL在5.7版本后默认安装了密码强度验证插件validate_password。您可以登录MySQL后查看密码强度规则。

    ```shell
    show variables like "%password%";
    ```

    本教程中创建新用户`user`，新用户密码为`PASSword123.`。

    4. 赋予用户对数据库`wordpress`的全部权限。

    ```shell
    grant all privileges on wordpress.* to 'user'@'localhost' identified by 'PASSword123.';
    ```

    5. 使配置生效。

    ```shell
    flush privileges;
    ```

    6. 退出MySQL。

    ```shell
    exit;
    ```

2. 下载WordPress，并移动至网站根目录。

    1. 下载WordPress。
    通过yum命令下载的WordPress保存在/usr/share/wordpress目录下。

    ```shell
    sudo yum -y install wordpress
    ```

    2. 将下载的WordPress移动至网站根目录。

    ```shell
    sudo mv /usr/share/wordpress /usr/share/nginx/html/wordpress
    ```

3. 修改WordPress配置文件。

    1. 进入移动后的WordPress路径下，软链接配置文件`wp-config.php`。

    ```shell
    cd /usr/share/nginx/html/wordpress
    sudo ln -snf /etc/wordpress/wp-config.php wp-config.php
    ```

    2. 编辑wp-config.php文件。

    ```shell
    sudo vim wp-config.php
    ```

    3. 按 `i` 键切换至编辑模式。
    根据已配置的WordPress数据库信息，修改MySQL相关配置信息，修改代码如下所示。
    WordPress网站的数据信息将通过数据库的`user`用户保存在名为`wordpress`的数据库中。

    ```shell
    // ** MySQL 设置 - 具体信息来自您正在使用的主机 ** //
    /** WordPress数据库的名称 */
    define('DB_NAME', 'wordpress');

    /** MySQL数据库用户名 */
    define('DB_USER', 'user');

    /** MySQL数据库密码 */
    define('DB_PASSWORD', 'PASSword123.');

    /** MySQL主机 */
    define('DB_HOST', 'localhost');
    ```

    4. 按`Esc`键，输入`:wq`后按`Enter`键，保存退出配置文件。

4. 修改Nginx配置文件。

    1. 运行以下命令，打开Nginx配置文件。

    ```shell
    sudo vim /etc/nginx/nginx.conf
    ```

    2. 按 `i` 键进入编辑模式。

        - 在`server`大括号内，将`root`后的内容替换为WordPress根目录。
        - 在`location ~ .php$`大括号内，将root后的内容替换为WordPress根目录。本示例中根目录为`/usr/share/nginx/html/wordpress`。

        [![piv5Xng.png](https://s11.ax1x.com/2024/01/04/piv5Xng.png)](https://imgse.com/i/piv5Xng)

    3. 按`Esc`键，输入`:wq`后按`Enter`键，保存退出配置文件。

    4. 运行以下命令，重启Nginx服务。

    ```shell
    sudo systemctl restart nginx
    ```

5. 安装并登录WordPress网站。

    1. 在本地物理机上使用浏览器访问http://ECS实例公网IP，进入WordPress安装页面。

    2. 填写网站基本信息，然后单击安装WordPress。
    填写信息参数说明：

        - 站点标题：WordPress网站的名称。例如：demowp。

        - 用户名：登录WordPress时所需的用户名，请注意安全性。例如：testwp。

        - 密码：登录WordPress时所需的密码，建议您设置安全性高的密码。

        - 您的电子邮件：用于接收通知的电子邮件。例如：username@example.com。

    3. 单击登录。

    4. 输入在安装WordPress时设置的用户名`testwp`和密码`Wp.123456`，然后单击 __登录__。成功进入您个人的WordPress网站。