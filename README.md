# Lnmp+Swoole
配置Lnmp和安装Swoole
# 准备条件
虚拟机下安装centos7(官网直接下镜像)
## 目标一：接通网络（NAT模式下）
* （1）新安装的centos需要配置ip地址、子网掩码、网关和DNS，执行以下命令打开文件进行配置
```php
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
* （2）然后对应修改或者添加以下语句
```php
BOOTPROTO="static" #静态连接
ONBOOT="yes" #系统启动时激活网卡
IPADDR=”192.168.8.130” #ip地址
NETMASK=”255.255.255.0” #子网掩码
GATEWAY=192.168.8.2 #网关
NM_CONTROLLED=no #修改后无需要重启网卡立即生效
DNS1=8.8.8.8
```
    注意：这里的ip地址、子网掩码、网关并不是随便填写  
    可以查看虚拟机的编辑->虚拟网络编辑器->DHCP配置，ip可以从起始ip到结束ip随便选一个，子网掩码就是对应的子网掩码
    网关则是虚拟机的编辑->虚拟网络编辑器->NAT配置，查看对应的网关
* （3）然后保存退出
```php 
:wq
```
* （4）然后重启网络服务
```php
service network restart
```
* （5）最后就可以尝试ping自己的服务器，也可以尝试ping百度的域名，如果可以则说明网络服务已经配置
## 目标二：安装php7(使用编译的方式安装)
* （1）安装vim编辑器
```php
yum install vim
```
* （2）安装wget下载器
```php
yum install wget
```
* （3）到php官网获取下载链接 以下例子是php7.1.15
```php
wget http://cn2.php.net/get/php-7.1.15.tar.gz/from/this/mirror
```
* （4）解压php压缩文件
```php
tar -zxvf mirror
```
* （5）安装其他扩展
```php
yum install gcc gcc++ libxml2-devel
```
* （6）检查环境配置(这里的ph7-legend是我准备安装的php目录)
```php
./configure --prefix=/usr/local/php7-legend --enable-fpm
```
* （7）编译安装依次执行以下代码
```php
make
```
```php
make install
```
* （8）配置环境变量依次执行
```php
vim /etc/profile
```
```php
PATH=$PATH:/usr/local/php/bin
export PATH
这两句加在末尾即可，保存退出再执行下面命令
```
```php
source /etc/profile
```
* （9）回到根目录，把php-7.1.15的php-development复制到/usr/local/php7-local/lib
```php
cp php-7.1.15/php-development /usr/local/php7-local/lib/php.ini
```
* （10）安装成功后就可以回到根目录，尝试新建一个php文件并执行
```php
vim test.php
```
```php
/usr/local/php7-legend/bin/php test.php
```
## 目标三：安装mysql
* （1）下载mysql的repo源
```php
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```
* （2）安装mysql的rpm包
```php
rpm -ivh mysql-community-release-el7-5.noarch.rpm
```
* （3）安装mysql
```php
yum install -y  mysql-server
```
* （4）更改mysql用户权限
```php
chown -R root:root /var/lib/mysql
```
* （5）重启服务：
```php
systemctl restart mysql.service
```
* （6）登陆
```php
mysql -u root
```
* （7）依次执行可修改密码
```php
use mysql;
```
```php
update user set password=password('123123') where user='root';
```
```php
exit;
```
# 目标四：安装nginx(使用编译的方式安装)
* （1）安装pcre扩展
```php
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.41.tar.gz
```
* （2）解压文件
```php
tar -zxvf pcre-8.41.tar.gz
```
* （3）进入pcre目录并检查环境配置
```php
cd pcre-8.41
```
```php
./configure --prefix=/usr/local/pcre-8.41
```
* （4）依次使用以下命令编译
```php
make
```
```php
make install
```
* （5）然后下载nginx
```php
wget http://nginx.org/download/nginx-1.12.2.tar.gz
```
* （6）解压nginx压缩包
```php
tar -zxvf nginx-1.12.2.tar.gz
```
* （7）进入nginx目录并检查环境配置
```php
cd nginx-1.12.2
```
```php
./configure --prefix=/usr/local/nginx --with-pcre=../pcre-8.41/
```
* （8）依次使用以下命令编译
```php
make
```
```php
make install
```
* （9）进入nginx目录开启服务
```php
cd /usr/local/nginx/sbin
```
```php
./nginx
```
* （10）在浏览器访问192.168.8.140会提示安装成功
* （11）如果访问不成功可以尝试关闭防火墙
```php
systemctl stop firewalld.service
```
# 目标五：配置通过nginx服务器访问php文件
* （1）进入php的fpm安装目录修改php-fpm配置
```php
cd /usr/local/php7-legend/etc
```
* （2）更改fpm文件名以conf结尾
```php
mv php-fpm.conf.default php-fpm.conf
```
* （3）进入php-fpm.d文件夹
```php
cd /usr/local/php7-legend/etc/php-fpm.d
```
* （4）复制一份配置文件
```php
cp www.conf.default www.conf
```
* （5）进入sbin目录启动php-fpm服务
```php
cd /usr/local/php7-legend/sbin
```
```php
./php-fpm
```
* （6）进入nginx配置目录
```php
cd /usr/local/nginx/conf
```
* （7）打开配置文件并在server内加上以下配置命令 保存退出
```php
vim nginx.conf
```
```php
location ~ \.php{
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_index /index.php;

                include /usr/local/nginx/conf/fastcgi_params;

                fastcgi_split_path_info         ^(.+\.php)(/.+)$;
                fastcgi_param PATH_INFO         $fastcgi_path_info;
                fastcgi_param PATH_TRANSLATED   $document_root$fastcgi_path_info;
                fastcgi_param SCRIPT_FILENAME   $document_root$fastcgi_script_name;
}
```
```php
:x
```
* （8）最后在nginx的html目录下添加测试php文件
```php
cd /usr/local/nginx/html
```
```php
vim test.php
```
* （9）在浏览器访问192.168.8.140/test.php即可（这里的ip记得修改）
# 目标六：安装swoole
* （1）首先下载安装必要的扩展
```php
yum install php-pear php-devel httpd
```
* （2）然后执行以下命令一键下载安装swoole
```php
pecl install swoole
```
* （3）然后再进入php.ini加上扩展
```php 
vim /etc/php.ini
```
```
extension=swoole.so
```
* （4）然后可以通过以下命令查看是否有swoole扩展
```php
php -m
```
* （5）然后在nginx下的html目录创建一个php文件编写swoole代码测试
```php
vim /usr/local/nginx/html/server.php
```
```php
$serv = new swoole_server("0.0.0.0", 9501);
  $serv->on('connect', function ($serv, $fd){
    echo "建立连接\n";
  });
  $serv->on('receive', function ($serv, $fd, $from_id, $data) {
    echo "接收到数据:{$data}";
    $serv->send($fd, 'Swoole: '.$data);
  });
  $serv->on('close', function ($serv, $fd) {
    echo "关闭连接\n";
  });
  $serv->start();
```
* （6）接着推荐下载一款测试软解--网络调试助手
下载好后协议选择TCP协议，服务器ip就是虚拟机最开始配置的ip地址，服务器端口可以填9501
* （7）然后是在centos下执行server.php文件
```php
/usr/local/php7-legend/bin/php /usr/local/nginx/html/server.php
```
最后在网络调试助手选择《连接》，如果服务器端提示建立连接则说明Lnmp+Swoole配置成功
