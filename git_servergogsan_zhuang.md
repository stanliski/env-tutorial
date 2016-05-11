# Git Server－Gogs安装

安装过程分为这些步骤：
- 新建用户；
- 下载源码编译 / 下载预编译二进制打包；
- 运行安装；
- 配置调整；
- 配置 nginx 反向代理；
- 保持服务运行；

> 注意，这里默认你已经安装好了 MySQL 服务器和 apache，如果没有，请自行查找如何安装和配置这些依赖。当然你也可以使用 SQLite 数据库。


#### 新建用户
`Gogs` 默认以 `git` 用户运行（你应该也不会想一个能修改 ssh 配置的程序以 root 用户运行吧？）。
运行 `sudo adduser git` 新建好 git 用户。
`su git` 以 git 用户登录，到 git 用户的主目录中新建好 .ssh 文件夹。

#### 下载解包

我使用的是预编译的二进制包。需要从源码编译的话，请参考一般 Go 语言项目的编译。下载后解包到你喜欢的地方，例如 `/usr/share/gogs/` 或者 `/opt/gogs/`。文件夹的内容如下

```bash
$ ls /home/git/gogs/
gogs  LICENSE  log  public  README.md  README_ZH.md  scripts  templates
```

#### 运行安装

  首先建立好数据库。在 `Gogs` 目录的 `scripts/mysql.sql` 文件是数据库初始化文件。执行 `mysql -u root -p < scripts/mysql.sql` （需要输入密码）即可初始化好数据库。

然后登录 `MySQL` 创建一个新用户 `gogs`，并将数据库 gogs 的所有权限都赋予该用户。

```sql
$ mysql -u root -p
> # （输入密码）
> create user 'gogs'@'localhost' identified by '密码';
> grant all privileges on gogs.* to 'gogs'@'localhost';
> flush privileges;
> exit;
```

运行 `gogs web` 把 `Gogs` 运行起来，然后访问` http://服务器IP:3000/` 来进行安装，填写好表单之后提交就可以了。
> 需要注意的是，0.6.9.0903 Beta 版本有个 bug，允许在关闭注册的情况下不添加管理员，这样安装完成之后将没有任何用户可以登录。所以请务必在安装界面指定一个管理员帐号。

#### 配置调整

配置文件位于 Gogs 目录的 custom/conf/app.ini，是 INI 格式的文本文件。详细的配置解释和默认值请参考官方文档，其中关键的配置大概是下面这些。

- `RUN_USER` 默认是 git，指定 Gogs 以哪个用户运行
- `ROOT` 所有仓库的存储根路径
- `PROTOCOL` 如果你使用 nginx 反代的话请使用 http，如果直接裸跑对外服务的话随意
- `DOMAIN` 域名。会影响 SSH clone 地址
- `ROOT_URL` 完整的根路径，会影响访问时页面上链接的指向，以及 HTTP clone 的地址
- `HTTP_ADDR` 监听地址，使用 nginx 的话建议 127.0.0.1，否则 0.0.0.0 也可以
-` HTTP_PORT` 监听端口，默认 3000
- `INSTALL_LOCK` 锁定安装页面
- `Mailer 相关的选项`. 其中，Mailer 可以使用 Mailgun 的免费邮件发送服务，将 Mailgun 的 SMTP 配置填入到配置中就好。

#### Apache 反代
编辑`/etc/httpd/conf/httpd.conf`, 添加如下内容

```xml
<VirtualHost *:80>
        ServerAdmin webmaster@localhost

        ServerAlias git.c4fun.cn
        ProxyPreserveHost On
        ProxyRequests Off

        <Proxy *>
                AddDefaultCharset off
                Order deny,allow
                Allow from all
        </Proxy>

        ProxyPass / http://127.0.0.1:3000/
        ProxyPassReverse / http://127.0.0.1:3000/
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### 重启服务

重启Gogs服务, 让其在后台默认运行

```
nohup ./gogs web &
```

重启Apache服务器

```bash
service httpd restart
```