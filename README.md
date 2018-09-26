## 介绍

一个加入 **支付宝面对面付** 的 ss panel 版本。[源项目地址](https://github.com/huazhipeng/ss-panel-v3-mod/)

## 环境

1.PHP 5.6 + (推荐 PHP 7.1.1)
2.MYSQL 5.5 + 
3.以及安装和配置过程中涉及到的种种东西。

## 安装
前端搭建：[安装魔改前端](https://github.com/huazhipeng/ss-panel-v3-mod-with-f2fpay/wiki/安装魔改前端)


前端搭建
搭建环境
centos 7

配置环境：安装lnmp1.4
下载并安装LNMP一键安装包
添加虚拟主机，可根据需求，添加 ssl 支持。

yum install screen -y
screen -S lnmp
wget -c http://soft.vpser.net/lnmp/lnmp1.4.tar.gz && tar zxf lnmp1.4.tar.gz && cd lnmp1.4 && ./install.sh lnmp
lnmp vhost add

移除防跨目录移除工具
该工具可以快速的移除防跨目录的限制

cd lnmp1.4/tools && ./remove_open_basedir_restriction.sh
按提示输入虚拟主机目录 /home/wwwroot/你的域名 回车确认即可。

开启scandir()函数
sed -i 's/,scandir//g' /usr/local/php/etc/php.ini
修改conf
vi /usr/local/nginx/conf/vhost/你的域名.conf

添加这段到server

location / 
{
	try_files $uri $uri/ /index.php$is_args$args;		                
}
修改root行

root /home/wwwroot/你的域名/public;
示例 conf

server
    {
        listen 80;
        server_name 你的域名;
        return 301 https://$host$request_uri;
    }
server
    {
        listen 443 ssl;
        server_name 你的域名;
        ssl_certificate /etc/letsencrypt/live/你的域名/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/你的域名/privkey.pem;
        index index.html index.htm index.php default.html default.htm default.php;
        root  /home/wwwroot/你的域名/public;

        include other.conf;
        #error_page   404   /404.html;
        include enable-php.conf;

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
            expires      12h;
        }
location ~ /\.
        {
            deny all;
        }
location / {
                        try_files $uri $uri/ /index.php$is_args$args;
                }
 access_log  /home/wwwlogs/你的域名.log;
安装面板程序
下载面板程序
cd /home/wwwroot/你的域名
yum install git -y
git clone -b master https://github.com/zyl6698/ss-panel-v3-mod-with-f2fpay.git tmp && mv tmp/.git . && rm -rf tmp && git reset --hard
chown -R root:root *
chmod -R 755 *
chown -R www:www storage
php composer.phar install
mv tool/alipay-f2fpay vendor/
mv -f tool/autoload_classmap.php vendor/composer/
配置数据库
登陆数据库

mysql -u root -p                                       // 这里需要输入密码
mysql>CREATE DATABASE database_name;                   //新建数据库
mysql>use database_name;                               // 选择数据库
mysql>source /home/wwwroot/你的域名/sql/glzjin_all.sql  // 导入.sql文件
配置 sspanel
cd /home/wwwroot/你的域名
cp config/.config.php.example config/.config.php
vi config/.config.php
lnmp restart
着重介绍面对面支付设置部分

$System_Config['payment_system']='f2fpay';
 #alipay,f2fpay
$System_Config['f2fpay_app_id']='';               
$System_Config['f2fpay_p_id']='';
$System_Config['alipay_public_key']='';
$System_Config['merchant_private_key']='';
第一个位置： $System_Config[‘f2fpay_app_id’]=” 这个是支付宝商家平台里的APPID，开通收款码服务后进入支付宝商家平台签约管理，查看PID和Key https://openhome.alipay.com/platform/keyManage.htm 有一个基础应用，就是那个APPID

第二个位置： $System_Config[‘f2fpay_p_id’] 这个是收款的支付宝账号，用来确认阿里消息正确性的。 https://openhome.alipay.com/platform/keyManage.htm?keyType=partner 签约管理里合作伙伴身份PID

第三个位置： $System_Config[‘alipay_public_key’] 指的是这里的支付宝公钥，注意是支付宝公钥。

第四个位置： $System_Config[‘merchant_private_key’] 这个是你自己的私钥。

创建管理员并同步用户
php xcat createAdmin          //创建管理员
php xcat syncusers            //同步用户
php xcat initQQWry            //下载IP解析库
php xcat resetTraffic         //重置流量
设置定时任务
执行 crontab -e命令, 添加以下五段

30 22 * * * php /home/wwwroot/站点文件夹/xcat sendDiaryMail 
*/1 * * * * php /home/wwwroot/站点文件夹/xcat synclogin
*/1 * * * * php /home/wwwroot/站点文件夹/xcat syncvpn
0 0 * * * php -n /home/wwwroot/站点文件夹/xcat dailyjob
*/1 * * * * php /home/wwwroot/站点文件夹/xcat checkjob    
*/1 * * * * php -n /home/wwwroot/站点文件夹/xcat syncnas


后端搭建：[安装魔改后端](https://github.com/huazhipeng/ss-panel-v3-mod-with-f2fpay/wiki/安装魔改后端)
安装魔改后端
理论上 Ubuntu 可以完全参照 Debian 的教程，如有问题请自行 debug

1、依赖安装
CentOS:

yum install epel-release -y
yum install python-pip git wget -y
Feroda:

dnf install git wgetpython-pip -y
Debian:

apt install git wget python-setuptools -y
easy_install pip
2、libsodium 安装
Fedora: dnf install libsodium -y

↑其实 Fedora 这样就够了(不过版本得跟随源，大多数情况比楼下旧

CentOS/Fedora: yum -y groupinstall "Development Tools"

Debian: apt install build-essential -y

接着编译安装

wget https://github.com/jedisct1/libsodium/releases/download/1.0.16/libsodium-1.0.16.tar.gz
tar xf libsodium-1.0.16.tar.gz && cd libsodium-1.0.16
./configure && make -j2 && make install
echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
ldconfig
cd ../ && rm -rf libsodium*
3、下载源代码并安装依赖
git clone -b manyuser https://github.com/esdeathlove/shadowsocks.git
cd shadowsocks
pip install -r requirements.txt
4、配置文件
cp apiconfig.py userapiconfig.py
cp config.json user-config.json
vi userapiconfig.py
配置文件说明如下：

# Config
#节点ID

NODE_ID = 1

#自动化测速，为0不测试，此处以小时为单位，要和 ss-panel 设置的小时数一致

SPEEDTEST = 6


#云安全，自动上报与下载封禁IP，1为开启，0为关闭

CLOUDSAFE = 1 

#自动封禁SS密码和加密方式错误的 IP，1为开启，0为关闭

ANTISSATTACK = 0

#是否接受上级下发的命令，如果你要用这个命令，请参考我(此处指 glzjin )之前写的东西，公钥放在目录下的 ssshell.asc

AUTOEXEC = 1

#单端口多用户设置，看重大更新说明

MU_SUFFIX = 'zhaoj.in'
MU_REGEX = '%5m%id.%suffix'

#不明觉厉

SERVER_PUB_ADDR = '127.0.0.1' # mujson_mgr need this to generate ssr link

#访问面板方式

API_INTERFACE = 'modwebapi' #glzjinmod (数据库方式连接)，modwebapi (http api)

#mudb，不要管
MUDB_FILE = 'mudb.json'

# HTTP API 的相关信息，看重大更新说明。

# 面板地址，区分https和http
WEBAPI_URL = 'https://zhaoj.in'

# 此处为.config.php中的muKey
WEBAPI_TOKEN = 'glzjin'

# Mysql 数据库连接信息

MYSQL_HOST = '127.0.0.1'

MYSQL_PORT = 3306

MYSQL_USER = 'ss'

MYSQL_PASS = 'ss'

MYSQL_DB = 'shadowsocks'

# 是否启用SSL连接，0为关，1为开

MYSQL_SSL_ENABLE = 0

# 客户端证书目录
MYSQL_SSL_CERT = '/root/shadowsocks/client-cert.pem'

MYSQL_SSL_KEY = '/root/shadowsocks/client-key.pem'

MYSQL_SSL_CA = '/root/shadowsocks/ca.pem'

# API，不用管

API_HOST = '127.0.0.1'

API_PORT = 80

API_PATH = '/mu/v2/'

API_TOKEN = 'abcdef'

API_UPDATE_TIME = 60

# Manager 不用管

MANAGE_PASS = 'ss233333333'

#if you want manage in other server you should set this value to global ip

MANAGE_BIND_IP = '127.0.0.1'

#make sure this port is idle

MANAGE_PORT = 23333
 

5、相关优化
修改文件句柄限制

cat >> /etc/security/limits.conf << EOF
* soft nofile 51200
* hard nofile 51200
EOF
然后执行

ulimit -n 51200

优化内核参数

cat >> /etc/sysctl.conf << EOF
fs.file-max = 51200
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
EOF
然后 sysctl -p

6、配置supervisord
CentOS/Fedora:

yum install supervisor -y
systemctl enable supervisord
wget https://raw.githubusercontent.com/zyl6698/ss-panel-v3-mod-with-f2fpay/master/tool/supervisord.conf -O /etc/supervisord.conf
systemctl start supervisord
Debian:

apt install supervisor -y
systemctl enable supervisor
wget https://raw.githubusercontent.com/zyl6698/ss-panel-v3-mod-with-f2fpay/master/tool/supervisord.conf -O /etc/supervisor/conf.d/supervisord.conf
systemctl start supervisor
默认仓库应该是 /root/shadowsocks ，如果不是，请自行修改 supervisord.conf 最底部 [program:mu] 中的 directory

关闭防火墙
systemctl disable firewalld
systemctl stop firewalld
其他事项
开启TCP Fast Open

确认服务器内核 > 3.7
确认内核选项 net.ipv4.tcp_fastopen 为 3.
user-config.json 里 fast_open 需要为 true.
## 交流联系
Telegram 群组：[https://t.me/modf2f](https://t.me/modf2f)


