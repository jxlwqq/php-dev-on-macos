2017 年 9 月 下旬，苹果公司正式发布了新一代 macOS，版本为 High Sierra (11.13)。
macOS High Sierra 预装了 Ruby(2.3.3)、PHP(7.1.7)、Perl(5.18.2)、Python(2.7.10) 等常用的脚本语言，以及 Apache(2.4.27)服务器。

#### 安装 iTerm2 

推荐 [iTerm2](http://www.iterm2.com/)，iTerm2 功能强大，可以替代系统默认的命令行终端。下载解压后，将 iTerm2 直接拖入"应用程序"目录。


#### 安装 PhpStorm

推荐 [JetBrains PhpStorm](https://www.jetbrains.com/phpstorm/) 作为集成开发工具。

#### 安装 Xcode

[Xcode](https://itunes.apple.com/cn/app/xcode/id497799835) 是苹果出品的包含一系列工具及库的开发套件。通过 AppStore 安装最新版本的 Xcode(9.0)。我们一般不会用 Xcode 来开发后端项目。但这一步也是`必须`的，因为 Xcode 会附带安装一些如 Git 等必要的软件。


#### 安装 Xcode Command Line Tools

这一步会帮你安装许多常见的基于 Unix 的工具。Xcode 命令行工具作为 Xcode 的一部分，包含了 GCC 编译器。在命令行中执行以下命令即可安装：

```
# 安装 Xcode Command Line Tools
xcode-select --install
```

当 Xcode 和 Xcode Command Line Tools 安装完成后，你需要启动 Xcode，并点击同意接受许可协议，然后关闭 Xcode 就可以了。这一步骤也是`必须`的，否则 Xcode 包含的一系列开发工具都将不可用。

#### 安装 Homebrew
 
[Homebrew](http://brew.sh/index_zh-cn.html) 作为 macOS 不可或缺的套件管理器，用来安装、升级以及卸载常用的软件。在命令行中执行以下命令即可安装：

```
# 使用系统自带的 ruby 安装 Homebrew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装后可以修改 Homebrew 源，国外源一直不是很给力，这里我们将 Homebrew 的 git 远程仓库改为[中国科学技术大学开源软件镜像](https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git)：

```
cd "$(brew --repo)"
git remote set-url origin git://mirrors.ustc.edu.cn/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```

#### 安装一些必要的工具包
```
brew install wget
brew install autoconf
brew install openssl
```

#### 安装 nginx

这里我们选择 nginx 代替系统自带的 Apache，作为我们的 Web 服务器：
```
brew install nginx
```

安装完成后，nginx 的一些常用命令：

```
sudo nginx # 启动 nginx 服务
nginx -h # nginx 帮助信息
sudo nginx -s stop|quit|reopen|reload # 停止|退出|重启|重载 nginx 服务
```

#### 安装 MySQL

推荐 MySQL 作为数据库服务器：
```bash
brew install mysql
```
当然，你也可以选择安装 PostgreSQL 或者 MariaDB。

安装完成后，启动 MySQL：
```bash
mysql.server start
```

进入 MySQL 服务器：
```bash
mysql -u root -p
```

#### 安装 Redis
推荐 Redis 作为 noSQL 数据库服务器：
```bash
brew install redis
```
安装完成后，启动 Redis：
```bash
redis-server
```

#### 安装 PHP 7.2

安装 PHP7.2 来代替系统自带的 PHP7.1：
```bash
brew tap homebrew/dupes  
brew tap homebrew/versions  
brew tap homebrew/homebrew-php
brew install php72
echo export PATH="$(brew --prefix homebrew/php/php72)/bin:$PATH" >> ~/.bash_profile # 代替系统自带的 php
source ~/.bash_profile
```
验证 PHP 以及 php-fpm 版本：
```bash
php -v
php-fpm -v
```
安装 php5.* 也是用类似方法。

#### 配置 nginx.conf 文件

通过以下命令可以查看 nginx.conf 文件的位置： 
```bash
nginx -h
```
输出：
```bash
nginx version: nginx/1.12.1
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/Cellar/nginx/1.12.1/)
  -c filename   : set configuration file (default: /usr/local/etc/nginx/nginx.conf)
  -g directives : set global directives out of configuration file
```
打开配置文件：
```bash
vi /usr/local/etc/nginx/nginx.conf
```
在文件末尾可以看到：
```bash
include servers/*;
```
它将同目录下的servers目录里的文件都包含了进来，由此，我们可以在servers文件里创建开发项目的配置信息：
```bash
cd servers/
vi test.conf
```
将以下配置信息，写入 test.conf文件中：
```nginx
server {
    listen 8099;
    server_name localhost;
    root /home/www/php_project;
    rewrite . /index.php;
    location / {
        index index.php index.html index.htm;
        autoindex on;
    }

    #proxy the php scripts to php-fpm
    location ~ \.php$ {
        include /usr/local/etc/nginx/fastcgi.conf;
        fastcgi_intercept_errors on;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

在上述的`/home/www/php_project`的目录下，我们创建一个 index.php 文件：
```bash
vi /home/www/php_project/index.php
```
写入内容：
```php
<?php
phpinfo();
```
重启 nginx:
```bash
sudo nginx -s stop && sudo nginx
```

打开浏览器，访问`localhost:8099`。可以看到关于 PHP 配置的信息。

#### 安装 Composer

Composer 是 PHP 用来管理依赖（dependency）关系的工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件。

安装并启动[国内镜像服务](https://pkg.phpcomposer.com/#how-to-use-packagist-mirror)：
```bash
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
composer config -g repo.packagist composer https://packagist.phpcomposer.com # 改为国内源
```

#### 安装 PHP 扩展

不推荐用 pecl 的方式安装 PHP 扩展。以 php-redis 扩展为例，下载源码包来进行安装：
```
wget https://pecl.php.net/get/redis-3.1.3.tgz # 下载源码包
tar -zxvf redis-3.1.3.tgz # 解压
cd redis-3.1.3
phpize # 生成编译配置                 
./configure # 编译配置检测
make # 编译
make install # 安装
```

扩展安装完成后，我们还需最后一步，修改`php.ini`文件，并重启 PHP-FPM：
```
# 追加 extension=redis.so
vi /usr/local/etc/php/7.2/php.ini

# 重启 php-fpm
sudo killall php-fpm && sudo php-fpm -D

# 查看是否安装成功
php -m |grep redis 
```