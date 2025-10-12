甲骨文ARM和AMD、亚马逊AWS Lightsail 以及大部分VPS通用DD方法 xhj009
系统安装ubuntu 20.04 (甲骨文) 或者Debian 10（AWS）
ssh登录系统后，输入sudo -i 切换到root用户。
运行以下命令更新系统并安装必要的软件
apt-get update -y && apt-get install curl wget vim dnsutils telnet -y
安装完成后，运行萌咖DD脚本：
bash <(wget --no-check-certificate -qO- 'https://raw.githubusercontent.com/MoeClub/Note/master/InstallNET.sh') -d 10 -v 64 -p "自定义root密码" -port "自定义ssh端口"
比如：

bash <(wget --no-check-certificate -qO- 'https://raw.githubusercontent.com/MoeClub/Note/master/InstallNET.sh') -d 10 -v 64 -p "password" -port "65001"

这样DD后就可以使用高位端口65001来登录vps了。 -d 10 代表DD的Debian 10系统。你可以 -d 12 来DD最新的Debian 12系统。
如果你DD其他系统可以看：
-firmware 额外的驱动支持

-d Debian系统 后面是系统版本号，例：9、10 ...

-c Centos系统 后面是系统版本号，例：6.9、6.10 ...

-u Ubuntu系统 后面是系统版本号，例：16.04、18.04 ...

-v 系统位数，64位或32位，只写数字

-a auto，全自动无人值守安装

-mirror 后面是指定镜像源地址

-p 后面写自定义密码

-ip-addr ifconfig -a 后获取到的 例：194.87.xxx.xxx

-ip-gate route -n 后获取到的 例 194.87.xxx.xxx

-ip-mask 255.255.xxx.xx

重点是谷歌云GCP的DD方法比较特殊。
谷歌云GCP我也是找了很多相关教程，失联了很多次，踩了很多坑。。。

现在给大家总结一下100%成功的方法。
首先在在谷歌云GCP后台使用ssh进入到vps里面
ssh登录系统后，输入sudo -i 切换到root用户。
运行以下命令更新系统并安装必要的软件
apt-get update -y && apt-get install curl wget vim dnsutils telnet -y
重点来了，直接按照甲骨文和AWS的方法去DD会导致VPS失联。按照我下面的方法一定成功！
首先在谷歌云GCP实例后台找到你VPS实例的内网地址和网关，如下：

大部分VPS以及甲骨文ARM/AMD、AWS、谷歌云GCP通用DD方法！

比如我的hk实例，内网地址就是10.170.0.4，那么网关就是10.170.0.1

然后将内网地址和网关填入DD命令里，子网掩码就是255.255.255.0 如下：
bash <(wget --no-check-certificate -qO- 'https://raw.githubusercontent.com/MoeClub/Note/master/InstallNET.sh') --ip-addr 10.170.0.4 --ip-gate 10.170.0.1 --ip-mask 255.255.255.0 -d 10 -v 64 -a -p "password" -port "65001" 