# Chamelenon
实现一种WiFi路由器伪装技术，代号变色龙chameleon，以便使变色龙附近的手机（STA,station)能自动关联认证，以使WiFi技术拥有更好的使用体验和更大的覆盖范围。

## 使用的开源项目
- OpenWrt
- hostapd
- wifidog

## 搭建方法
### PC上搭建过程

**硬件要求**：USB无线网卡
- 配置`hostapd`（`hostapd-PC`）。按照[这里](https://github.com/mengning/chameleon/wiki/Multiple-SSIDs-Configuration)的方法搭建`hostapd`的多`ssid`模式（使用两个`ssid`，一个是公有的`ssid`，另一个是为动态创建而准备的）并将配置文件换成本项目`config/PC/`目录下的`hostapd-phy0.conf`。
- 配置`wifidog`(`wifidog-1.2.1`)。进入`wifidog-1.2.1`,`make`编译。
    - 将`config/PC/`目录下的`wifidog.conf`复制到`/usr/local/etc/`目录。
    - 将`wifidog-1.2.1`目录下的`wifidog-msg.html`复制到`/var/www/html/`目录。

**运行**：

首先启动`hostapd`:
```sh
cd hostapd-PC/hostapd
sudo ./hostapd /etc/hostapd-phy0.conf
```
启动`wifidog`:
```sh
cd wifidog-1.2.1/src
sudo ./wifidog -f -d 7
```

### 路由器上搭建过程
- 下载`OpenWrt`源码。[OpenWrt 15.05 branch (Chaos Calmer)](https://dev.openwrt.org/wiki/GetSource)。**注意**：请保证磁盘至少有十几G的空闲空间，因为后续编译会下载大量软件包，空间太小会编译报错，提示空间不足。

- 首次编译OpenWrt。
    - 安装必要依赖包：</br>![](http://7xqbsh.com1.z0.glb.clouddn.com/OpenWrt依赖包.PNG)
    - 依赖安装完成后
    ```sh
    ./scripts/feeds update -a
    ./scripts/feeds install -a
    make defconfig
    make prereq
    make menuconfig
    ```
        - `Target System`选择`Atheros AR7xxx/AR9xxx`
        - `Target Profile`选择`TP-LINK TL_WR720N`
        - 选中`luci`。在`LuCI`->`Collections`->`luci`
        - 选中`wifidog`。在`Network`->`Captive Portals`->`wifidog`

    最后`make`。期间会下载大量的软件包，必须保持网络通常，有些软件包还需要翻墙下载。大概需要一两个小时左右，具体时间视网速而定。编译完成后，会在`bin`目录下生成相应的固件。

- 编译修改后的源码。查看目录`OpenWrt/dl`目录下`hostapd`和`wifidog`压缩包的名称，将本项目中的`hostapd-OpenWrt`和`wifidog-1.2.1`分别打包压缩成对应的相同文件名压缩包，注意，压缩包解压后的文件夹名称和原来的压缩包解压出的文件夹也要相同。打包完成后，替换原来`OpenWrt/dl`目录下`hostapd`和`wifidog`压缩包。删除`package/network/services/hostapd/patches`目录下的所有`patch`文件，因为本项目中修改的`hostapd`已经打过`patch`,再次打`patch`会报错。然后重新`make`。

- 烧写至路由器。编译完成后，将`bin/ar71xxx/`目录下的`openwrt-ar71xx-generic-tl-wr720n-v3-squashfs-sysupgrade.bin`烧进路由器，烧写方法[参考这里](http://www.right.com.cn/forum/thread-41910-1-1.html)，也可以使用`LuCI`界面自带的固件升级方法来烧写。

- 配置路由器。
    - 设置登录密码，启动wifi，此时有个系统默认配置的wifi,点击编辑，将wifi名改为`wifi2`，并为其设置一个`WPA-PSK`模式的密码。此wifi网络是作为动态创建使用的。
    - 利用`scp`将本项目`config/OpenWrt/`目录下的`wifidog.conf`拷贝至路由器的`/etc/`目录。
    - 参考[这里](https://wiki.openwrt.org/doc/recipes/routedap)，为路由器创建一个非桥接模式的网络接口wifi。
    - 新建一个wifi网络，默认名为OpenWrt，在其`General Setup`里面将`Network`改为上一步新建的接口wifi。此wifi网络即为公有的wifi。
    - 拔掉电源，重启路由器。


## 使用
首次使用，打开手机wifi，连接公有的OpenWrt，打开浏览器访问任意网页，产生页面跳转，注册用户名密码。注册成功后，断开当前公有wifi，刷新无线网络，将会看到为你动态创建的wifi网络，名称和密码就是你刚才注册使用的用户名和密码，连接后即可正常上网。以后使用时，不管到哪里，只要对方使用的是这款路由器，都会为你动态创建一个以你用户名命名的wifi，手机将会自动连接，不受地理范围的限制。


## Server
Server.zip 为服务器代码，解压以后可以部署到服务器上
 - 服务器需要安装的依赖为 ：
 - Mysql
 - JDK 1.6 ＋
 - Tomcat 7 ＋
 - 内存512M ＋
 - 以及linux 环境下的 各种build－essential
 - CPU 单核可运行
服务器可以直接部署到tomcat下，数据库的 root 和 psw 参见代码
服务器可添加Nginx 作为反向代理做分流，以Apache 作为后端服务
