# OpenWrt firmware for Newifi D2

Auto build OpenWrt firmware for Newifi D2 via GitHub Actions

自动化编译Newifi D2.

1、本地环境准备
增加非root用户并设置密码及权限
L大在前面说清楚了，不允许使用root用户进行编译，那么我们听从指挥，直接用非root用户进行下面所有的操作。

相关的命令如下：

useradd bozai #bozai为你建立的用户名
passwd bozai #为用户bozai设置密码

设置完毕密码后，我们需要对你刚才建立的用户赋予权限

在SSH工具里面，找到以下文件 /etc/sudoers ，双击打开。
找到 root	ALL=(ALL:ALL) ALL，并在后面加入一行，写入刚才你建立的用户名：
bozai ALL=(ALL:ALL) ALL

切换为刚才新建的用户（bozai），命令： su bozai

2、升级系统，并安装必要组件
输入命令sudo apt-get update，并输入密码执行升级系统。
运行以下命令安装必要组件

sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3.5 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf

因为依赖的组件很多，若你不确定上述组件是否正确被安装，可以安装重复几次。若是提示如下信息，请YES。
----------------------------Configuring libsl1.1:cmd64---------------------------------
There are services installed on your system which need to be restarted when certain libraries, such as libpam,
libc, and libssl, are upgraded. Since these restarts may cause interruptions of service for the system, you
will normally be prompted on each upgrade for the list of services you wish to restart. You can choose this
option to avoid being prompted; instead, all necessary restarts will be done for you automatically so you can
avoid being asked questions on each library upgrade.
Restart services during package upgrades without asking?
        <Yes>                      <No>

3、下载编译源代码
为了避开很多人下载源码的时候出问题，我们先对LEDE文件夹赋予权限了以后再下载。分别运行以下四条代码：
（或许会提示输入密码，请输入当前非root用户的用户的密码）

cd /
sudo mkdir lede
sudo chmod 777 lede
git clone https://github.com/coolsnowwolf/lede

然后 cd lede 进入LEDE文件夹 （千万别忽略这一步）

4、设定并更新软件包
PS：若你是想在固件里面拥有SSR PLUS或是其他VPN插件，请在SSH工具里面找到如下文件 /lede/feeds.conf.default ，双击打开。

把 src-git helloworld https://github.com/fw876/helloworld 前面的 # 去掉

你也可以在网上搜寻其他的 src-git 地址，用于集成其他的各种插件。

推荐下面的软件包，几乎涵盖了你需要插件
src-git kenzo https://github.com/V2RaySSR/openwrt-packages
src-git small https://github.com/V2RaySSR/small

控制台输入以下命令

cd /
cd lede
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig

找到 LUCI ，回车，然后找到 Applications 回车，选择你需要选择的各种插件。
选择完毕以后，按 ESC 退出，提示 YES 就 YES。

5、下载编译DL库（全局模式）
make -j8 download V=s 下载dl库（国内请尽量全局科学上网）

这一部分关键取决于你的节点或是VPS的速度，请耐心等待。

PS : 这一步下载的东西很多。不确定是否全部下载成功，可重复运行N次代码。直到没有出现下载的状态。

6、开始编译固件
输入 make -j1 V=s 开始编译固件

7、编译完成
大概4个小时以后（看机器性能）可以在SSH工具如下目录找到编译的固件 /lede/bin/targets

第二次编译
什么意思？因为第一次编译我们下载了很多必要的组件，所以，你若是以后再想编译，就很快了，也简单了。

切换为新建的非ROOT用户，输入以下命令

cd /
cd lede
git pull
若你是想在固件里面拥有SSR PLUS或是其他VPN插件，请找到如下文件 /lede/feeds.conf.default

把 src-git helloworld https://github.com/fw876/helloworld 前面的 # 去掉

然后输入命令

./scripts/feeds update -a && ./scripts/feeds install -a
make defconfig
make -j8 download
make -j$(($(nproc) + 1)) V=s

-----------------备注：---------------------------------------------------------------------
make menuconfig  进入定制界面
进入编译选项配置界面,.按照需要配置.( ‘*’ 代表编入固件，‘M’ 表示编译成模块或者IPK包， ‘空’不编译 )

非常感谢大佬”L有大雕“更正补充，20181121
选择LuCI 配置 添加插件应用：常用
—————————————————————————————–
LuCI —> Applications —> luci-app-accesscontrol  #访问时间控制
LuCI —> Applications —> luci-app-adbyby-plus   #广告屏蔽大师Plus +
LuCI —> Applications —> luci-app-arpbind  #IP/MAC绑定
LuCI —> Applications —> luci-app-autoreboot  #支持计划重启
LuCI —> Applications —> luci-app-ddns   #动态域名 DNS（集成阿里DDNS客户端）
LuCI —> Applications —> luci-app-filetransfer  #文件传输（可web安装ipk包）
LuCI —> Applications —> luci-app-firewall   #添加防火墙
LuCI —> Applications —> luci-app-flowoffload  #Turbo ACC网络加速（集成FLOW,BBR,NAT,DNS…
LuCI —> Applications —> luci-app-frpc   #内网穿透 Frp
LuCI —> Applications —> luci-app-guest-wifi  #WiFi访客网络
LuCI —> Applications —> luci-app-ipsec-virtual**d  #virtual**服务器 IPSec
LuCI —> Applications —> luci-app-mwan3   #MWAN3负载均衡
LuCI —> Applications —> luci-app-mwan3helper   #MWAN3分流助手
LuCI —> Applications —> luci-app-nlbwmon   #网络带宽监视器
LuCI —> Applications —> luci-app-ramfree  #释放内存
LuCI —> Applications —> luci-app-samba   #网络共享（Samba）
LuCI —> Applications —> luci-app-sqm  #流量智能队列管理（QOS）
——————————————————————————————-
LuCI —> Applications —> luci-app-乳酸菌饮料-plus   #乳酸菌饮料低调上网Plus+
luci-app-乳酸菌饮料-plus —> Include s-s New Versiong  #新SS代理
luci-app-乳酸菌饮料-plus —> Include s-s Simple-obfs Plugin  #simple-obfs简单混淆工具   *
luci-app-乳酸菌饮料-plus —> Include s-s v贰瑞 Plugin  #SS v贰瑞插件   *
luci-app-乳酸菌饮料-plus —> Include v贰瑞  #v贰瑞代理
luci-app-乳酸菌饮料-plus —> Include Trojan  #Trojan代理
luci-app-乳酸菌饮料-plus —> Include red—socks2  #red—socks2代理   *
    luci-app-乳酸菌饮料-plus —> Include Kcptun  #Kcptun加速
luci-app-乳酸菌饮料-plus —> Include 违禁软件 Server  #乳酸菌饮料服务器
——————————————————————————————-
LuCI —> Applications —> luci-app-syncdial  #多拨虚拟网卡（原macvlan）
LuCI —> Applications —> luci-app-unblockmusic  #解锁网易云灰色歌曲3合1新版本
LuCI —> Applications —> luci-app-upnp   #通用即插即用UPnP（端口自动转发）
LuCI —> Applications —> luci-app-vlmcsd  #KMS服务器设置
LuCI —> Applications —> luci-app-vsftpd  #FTP服务器
LuCI —> Applications —> luci-app-wifischedule  #WiFi 计划
LuCI —> Applications —> luci-app-wirele违禁软件egdb  #WiFi无线
LuCI —> Applications —> luci-app-wol   #WOL网络唤醒
LuCI —> Applications —> luci-app-wrtbwmon  #实时流量监测
LuCI —> Applications —> luci-app-xlnetacc  #迅雷快鸟
LuCI —> Applications —> luci-app-zerotier  #ZeroTier内网穿透
Extra packages  —>  ipv6helper  #支持 ipv6
Utilities  —>  open-vm-tools  #打开适用于VMware的VM Tools

以下是全部：                               注：应用后面标记 “ * ” 为最近新添加
—————————————————————————————–
LuCI —> Applications —> luci-app-accesscontrol  #访问时间控制
LuCI —> Applications —> luci-app-acme  #ACME自动化证书管理环境
LuCI —> Applications —> luci-app-adblock   #ADB广告过滤
LuCI —> Applications —> luci-app-adbyby-plus  #广告屏蔽大师Plus +
LuCI —> Applications —> luci-app-adbyby   #广告过滤大师（已弃）
LuCI —> Applications —> luci-app-adkill   #广告过滤（已弃）
LuCI —> Applications —> luci-app-advanced-reboot  #Linksys高级重启
LuCI —> Applications —> luci-app-ahcp  #支持AHCPd
LuCI —> Applications —> luci-app-aliddns   #阿里DDNS客户端（已弃，集成至ddns）
LuCI —> Applications —> luci-app-amule  #aMule下载工具
LuCI —> Applications —> luci-app-aria2 # Aria2下载工具
LuCI —> Applications —> luci-app-arpbind  #IP/MAC绑定
LuCI —> Applications —> luci-app-asterisk  #支持Asterisk电话服务器
LuCI —> Applications —> luci-app-attendedsysupgrade  #固件更新升级相关
LuCI —> Applications —> luci-app-autoreboot  #支持计划重启
LuCI —> Applications —> luci-app-baidupcs-web  #百度网盘管理
LuCI —> Applications —> luci-app-bcp38  #BCP38网络入口过滤（不确定）
LuCI —> Applications —> luci-app-bird1-ipv4  #对Bird1-ipv4的支持
LuCI —> Applications —> luci-app-bird1-ipv6  #对Bird1-ipv6的支持
LuCI —> Applications —> luci-app-bird4   #Bird 4（未知）（已弃）
LuCI —> Applications —> luci-app-bird6   #Bird 6（未知）（已弃）
LuCI —> Applications —> luci-app-bmx6  #BMX6路由协议
LuCI —> Applications —> luci-app-bmx7  #BMX7路由协议
LuCI —> Applications —> luci-app-caldav  #联系人（已弃）
LuCI —> Applications —> luci-app-cifsd  #网络共享CIFS/SMB服务器   *
LuCI —> Applications —> luci-app-cjdns  #加密IPV6网络相关
LuCI —> Applications —> luci-app-clamav  #ClamAV杀毒软件
LuCI —> Applications —> luci-app-commands   #Shell命令模块
LuCI —> Applications —> luci-app-cshark   #CloudShark捕获工具
LuCI —> Applications —> luci-app-ddns   #动态域名 DNS（集成阿里DDNS客户端）
LuCI —> Applications —> luci-app-diag-core   #core诊断工具
LuCI —> Applications —> luci-app-dnscrypt-proxy  #DNSCrypt解决DNS污染
LuCI —> Applications —> luci-app-dnsforwarder  #DNSForwarder防DNS污染
LuCI —> Applications —> luci-app-dnspod  #DNSPod动态域名解析（已弃）
LuCI —> Applications —> luci-app-dockerman  #Docker容器   *
LuCI —> Applications —> luci-app-dump1090  #民航无线频率（不确定）
LuCI —> Applications —> luci-app-dynapoint  #DynaPoint（未知）
LuCI —> Applications —> luci-app-e2guardian   #Web内容过滤器
LuCI —> Applications —> luci-app-familycloud   #家庭云盘
LuCI —> Applications —> luci-app-filetransfer  #文件传输（可web安装ipk包）
LuCI —> Applications —> luci-app-firewall   #添加防火墙
LuCI —> Applications —> luci-app-flowoffload  #Turbo ACC网络加速（集成FLOW,BBR,NAT,DNS…
LuCI —> Applications —> luci-app-freifunk-diagnostics   #freifunk组件 诊断（未知）
LuCI —> Applications —> luci-app-freifunk-policyrouting  #freifunk组件 策略路由（未知）
LuCI —> Applications —> luci-app-freifunk-widgets  #freifunk组件 索引（未知）
LuCI —> Applications —> luci-app-frpc   #内网穿透 Frp
LuCI —> Applications —> luci-app-fwknopd  #Firewall Knock Operator服务器
LuCI —> Applications —> luci-app-guest-wifi   #WiFi访客网络
LuCI —> Applications —> luci-app-gfwlist   #GFW域名列表（已弃）
LuCI —> Applications —> luci-app-haproxy-tcp   #HAProxy负载均衡-TCP
LuCI —> Applications —> luci-app-hd-idle  #硬盘休眠
LuCI —> Applications —> luci-app-hnet  #Homenet Status家庭网络控制协议
LuCI —> Applications —> luci-app-ipsec-virtual**d  #virtual**服务器 IPSec
LuCI —> Applications —> luci-app-kodexplorer  #KOD可道云私人网盘
LuCI —> Applications —> luci-app-kooldns  #virtual**服务器 ddns替代方案（已弃）
LuCI —> Applications —> luci-app-koolproxy  #KP去广告（已弃）
LuCI —> Applications —> luci-app-lxc   #LXC容器管理
LuCI —> Applications —> luci-app-meshwizard #网络设置向导
LuCI —> Applications —> luci-app-minidlna   #完全兼容DLNA / UPnP-AV客户端的服务器软件
LuCI —> Applications —> luci-app-mjpg-streamer   #兼容Linux-UVC的摄像头程序
LuCI —> Applications —> luci-app-mtwifi  #MTWiFi驱动的支持
LuCI —> Applications —> luci-app-mmc-over-gpio   #添加SD卡操作界面（已弃）
LuCI —> Applications —> luci-app-multiwan   #多拨虚拟网卡（已弃，移至syncdial）
LuCI —> Applications —> luci-app-mwan   #MWAN负载均衡（已弃）
LuCI —> Applications —> luci-app-mwan3   #MWAN3负载均衡
LuCI —> Applications —> luci-app-mwan3helper   #MWAN3分流助手
LuCI —> Applications —> luci-app-n2n_v2   #N2N内网穿透 N2N v2 virtual**服务
LuCI —> Applications —> luci-app-netdata  #Netdata实时监控（图表）
LuCI —> Applications —> luci-app-nft-qos  #QOS流控 Nftables版
LuCI —> Applications —> luci-app-ngrokc  #Ngrok 内网穿透（已弃）
LuCI —> Applications —> luci-app-nlbwmon   #网络带宽监视器
LuCI —> Applications —> luci-app-noddos  #NodDOS Clients 阻止DDoS攻击
LuCI —> Applications —> luci-app-nps  #内网穿透nps   *
LuCI —> Applications —> luci-app-ntpc   #NTP时间同步服务器
LuCI —> Applications —> luci-app-ocserv  #OpenConnect virtual**服务
LuCI —> Applications —> luci-app-olsr  #OLSR配置和状态模块
LuCI —> Applications —> luci-app-olsr-services  #OLSR服务器
LuCI —> Applications —> luci-app-olsr-viz   #OLSR可视化
LuCI —> Applications —> luci-app-openvirtual**  #Openvirtual**客户端
LuCI —> Applications —> luci-app-openvirtual**-server  #易于使用的Openvirtual**服务器 Web-UI
LuCI —> Applications —> luci-app-oscam   #OSCAM服务器（已弃）
LuCI —> Applications —> luci-app-p910nd   #打印服务器模块
LuCI —> Applications —> luci-app-pagekitec   #Pagekite内网穿透客户端
LuCI —> Applications —> luci-app-polipo  #Polipo代理(是一个小型且快速的网页缓存代理)
LuCI —> Applications —> luci-app-pppoe-relay  #PPPoE NAT穿透 点对点协议（PPP）
LuCI —> Applications —> luci-app-p p t p-server  #virtual**服务器 p p t p（已弃）
LuCI —> Applications —> luci-app-privoxy  #Privoxy网络代理(带过滤无缓存)
LuCI —> Applications —> luci-app-qbittorrent  #BT下载工具（qBittorrent）
LuCI —> Applications —> luci-app-qos   #流量服务质量(QoS)流控
LuCI —> Applications —> luci-app-radicale   #CalDAV/CardDAV同步工具
LuCI —> Applications —> luci-app-ramfree  #释放内存
LuCI —> Applications —> luci-app-rp-pppoe-server  #Roaring Penguin PPPoE Server 服务器
LuCI —> Applications —> luci-app-samba   #网络共享（Samba）
LuCI —> Applications —> luci-app-samba4   #网络共享（Samba4）
LuCI —> Applications —> luci-app-sfe  #Turbo ACC网络加速（flowoffload二选一）
LuCI —> Applications —> luci-app-s-s   #SS低调上网（已弃）
LuCI —> Applications —> luci-app-s-s-libes  #SS-libev服务端
LuCI —> Applications —> luci-app-shairplay  #支持AirPlay功能
LuCI —> Applications —> luci-app-siitwizard  #SIIT配置向导  SIIT-Wizzard
LuCI —> Applications —> luci-app-simple-adblock  #简单的广告拦截
LuCI —> Applications —> luci-app-smartdns  #SmartDNS本地服务器（已弃）
LuCI —> Applications —> luci-app-softethervirtual**  #SoftEther virtual**服务器  NAT穿透   *
LuCI —> Applications —> luci-app-splash  #Client-Splash是无线MESH网络的一个热点认证系统
LuCI —> Applications —> luci-app-sqm  #流量智能队列管理（QOS）
LuCI —> Applications —> luci-app-squid   #Squid代理服务器
——————————————————————————————————-
LuCI —> Applications —> luci-app-乳酸菌饮料-plus   #乳酸菌饮料低调上网Plus+
luci-app-乳酸菌饮料-plus —> Include s-s New Version  #新SS代理
luci-app-乳酸菌饮料-plus —> Include s-s Simple-obfs Plugin  #simple-obfs简单混淆工具   *
luci-app-乳酸菌饮料-plus —> Include s-s v贰瑞 Plugin  #SS v贰瑞插件   *
luci-app-乳酸菌饮料-plus —> Include v贰瑞  #v贰瑞代理
luci-app-乳酸菌饮料-plus —> Include Trojan  #Trojan代理
luci-app-乳酸菌饮料-plus —> Include red—socks2  #red—socks2代理   *
luci-app-乳酸菌饮料-plus —> Include Kcptun  #Kcptun加速
luci-app-乳酸菌饮料-plus —> Include 违禁软件 Server  #乳酸菌饮料服务器
luci-app-乳酸菌饮料-plus —> Include DNS2SOCKS  #DNS服务器（已弃）
luci-app-乳酸菌饮料-plus —> Include 违禁软件 Socks and Tunnel（已弃）
luci-app-乳酸菌饮料-plus —> Include Socks Server  #socks代理服务器（已弃）
——————————————————————————————————-
LuCI —> Applications —> luci-app-乳酸菌饮料-pro  #乳酸菌饮料-Pro（已弃）
LuCI —> Applications —> luci-app-乳酸菌饮料server-python  #违禁软件 Python服务器
LuCI —> Applications —> luci-app-statistics  #流量监控工具
LuCI —> Applications —> luci-app-syncdial  #多拨虚拟网卡（原macvlan）
LuCI —> Applications —> luci-app-tinyproxy  #Tinyproxy是 HTTP(S)代理服务器
LuCI —> Applications —> luci-app-transmission   #BT下载工具
LuCI —> Applications —> luci-app-travelmate  #旅行路由器
LuCI —> Applications —> luci-app-ttyd   #网页终端命令行
LuCI —> Applications —> luci-app-udpxy  #udpxy做组播服务器
LuCI —> Applications —> luci-app-uhttpd  #uHTTPd Web服务器
——————————————————————————————————-
LuCI —> Applications —> luci-app-unblockmusic  #解锁网易云灰色歌曲3合1新版本
UnblockNeteaseMusic Golang Version  #Golang版本   *
UnblockNeteaseMusic NodeJS Version  #NodeJS版本   *
——————————————————————————————————-
LuCI —> Applications —> luci-app-unblockneteasemusic-go  #解除网易云音乐（合并）
LuCI —> Applications —> luci-app-unblockneteasemusic-mini  #解除网易云音乐（合并）
LuCI —> Applications —> luci-app-unbound  #Unbound DNS解析器
LuCI —> Applications —> luci-app-upnp   #通用即插即用UPnP（端口自动转发）
LuCI —> Applications —> luci-app-usb-printer   #USB 打印服务器
LuCI —> Applications —> luci-app-v贰瑞-server   #v贰瑞 服务器
LuCI —> Applications —> luci-app-v贰瑞-pro  #v贰瑞透明代理（已弃，集成乳酸菌饮料）
LuCI —> Applications —> luci-app-verysync  #微力同步   *
LuCI —> Applications —> luci-app-vlmcsd  #KMS服务器设置
LuCI —> Applications —> luci-app-vnstat   #vnStat网络监控（图表）
LuCI —> Applications —> luci-app-virtual**bypass  #virtual** BypassWebUI  绕过virtual**设置
LuCI —> Applications —> luci-app-vsftpd  #FTP服务器
LuCI —> Applications —> luci-app-watchcat  #断网检测功能与定时重启
LuCI —> Applications —> luci-app-webadmin  #Web管理页面设置
LuCI —> Applications —> luci-app-webshell  #网页命令行终端（已弃）
LuCI —> Applications —> luci-app-wifischedule  #WiFi 计划
LuCI —> Applications —> luci-app-wireguard  #virtual**服务器 WireGuard状态
LuCI —> Applications —> luci-app-wirele违禁软件egdb  #WiFi无线
LuCI —> Applications —> luci-app-wol   #WOL网络唤醒
LuCI —> Applications —> luci-app-wrtbwmon  #实时流量监测
LuCI —> Applications —> luci-app-xlnetacc  #迅雷快鸟
LuCI —> Applications —> luci-app-zerotier  #ZeroTier内网穿透

—————————————————————————————————
支持 iPv6：
1、Extra packages  —>  ipv6helper  （选定这个后下面几项自动选择了）
Network  —>  odhcp6c
Network  —>  odhcpd-ipv6only
LuCI  —>  Protocols  —>  luci-proto-ipv6
LuCI  —>  Protocols  —>  luci-proto-ppp

2、打开适用于VMware的VM Tools
Utilities  —>  open-vm-tools

3、第二次编译：
cd lede                                                                               # 进入LEDE目录
git pull                                                                                # 同步更新大雕源码
./scripts/feeds update -a && ./scripts/feeds install -a               # 更新Feeds
rm -rf ./tmp && rm -rf .config                                               # 清除编译配置和缓存
make menuconfig                                                                # 进入编译配置菜单
make -jn V=99                                                                    # 开始编译 n=线程数+1，例如4线程的I5填-j5
