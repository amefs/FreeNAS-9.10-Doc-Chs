#FreeNAS 中文文档(9.10.2-U1)

FreeNAS作为一个基于FreeBSD的NAS系统, 稳定性较好, 有着较为完善的RAID支持, 同时使用ZFS文件系统, 有着不错的IO性能, 其功能也能够通过自带的插件扩展.

以下文档翻译自[FreeNAS®](http://doc.freenas.org "freenas.org")

###目录
* [简介](#简介)
	* [硬件要求](#硬件要求)
		* [RAM](#RAM)
		* [操作系统介质](#操作系统介质)
		* [存储介质与控制器](#存储介质与控制器)
		* [网络接口](#网络接口)
	* [安装与升级](#安装与升级)
		* [获取FreeNAS®](#获取FreeNAS®)
		* [准备安装介质](#准备安装介质)
		* [安装](#安装)
		* [升级](#升级)
	* [启动](#启动)
		* [获取一个IP](#获取一个ip)
		* [登录](#登录)
	* [账户](#账户)
		* [配置用户组](#配置用户组)
		* [配置账户](#配置账户)
	* [系统](#系统)
		* [信息](#信息)
		* [常规](#常规)
		* [启动器](#启动器)
			* [镜像启动设备](#镜像启动设备)
		* [高级](#高级)
			* [自动协调](#自动协调)
		* [电子邮件](#电子邮件)
		
##简介
###硬件要求

FreeNAS® 9.10.2-U1 基于 FreeBSD 10.3 开发, 支持[FreeBSD硬件兼容列表](http://www.freebsd.org/releases/10.3R/hardware.html "FreeBSD Hardware Compatibility List")中相同的硬件, [支持的处理器列表](https://www.freebsd.org/releases/10.3R/hardware.html#proc " 2.1 amd64").现在的FreeNAS系统只能运行于64位的处理器上.

*注意: FreeNAS®从GPT分区引导。 这意味着系统BIOS必须能够使用旧版BIOS固件(BIOS legacy)接口或EFI进行引导。*

####RAM

想要尽可能发挥出FreeNAS®系统的性能, 那么就应该尽可能多的安装RAM. 推荐的最小内存是8GB. 内存越多, FreeNAS®就能够发挥更好的性能.

因为以上的理由, 你的系统可能会需要更多的内存, 其中有一些通用的规则:

+ 多用户使用 Active Directory, 需要为winbind的内部缓存额外添加2GB RAM;

+ 使用iSCSI功能, 在要求不严苛的前提下需要最少安装16GB RAM, 或者最少安装32GB RAM来获得一个不错的性能;

+ 当FreeNAS® 安装于嵌入式系统中(没有外设的平台), 应当在BIOS设置中禁用与显卡分享内存;

+ 如果使用 ZFS deduplication(ZFS去重) 功能, 那么需要确保每TB的存储空间能够分配到5GB内存.

当硬件支持(CPU+主板芯片组), 同时有足够的资金, 那么建议安装ECC RAM. ECC内存在可以为ZFS的校验提供可靠的保障, 如果数据很重要, 那么更应该安装ECC内存.

当FreeNAS®的系统内存小于8GB时, 会降低性能, 同时有可能发生一些错误, 所以在使用FreeNAS®存储数据之前, 建议添加足够的内存.

####操作系统介质

FreeNAS® 应当被安装在一个与存储磁盘不同的介质上. 这个介质可以是U盘, 可以是SSD, 可以是嵌入式的闪存, 也可以是DOM盘, 不建议将系统安装到存储硬盘中, 安装系统的磁盘将不可以被用作存放数据.

*注意: 如果希望将FreeNAS®安装到U盘, 那么需要两个U盘, 一个是目标盘, 一个是包含了安装镜像的安装盘, 想要将FreeNAS®安装到存有安装镜像的U盘中是不可能的, 安装过程中必须注意选择正确的安装目标, 安装完毕移除USB设备后, 可能需要重新设置BIOS引导顺序*

在选择安装的目标介质时, 需要注意以下原则:

+ 最小需要的的可用介质大小为8GB, 这为操作系统和多个引导提供了空间, 由于每次更新都会创建一个引导环境, 因此建议最少要具有8GB存储空间. 32GB的空间就可以提供给更多的引导环境使用;

+ 如果想要创建自己的引导环境, 那么每一个引导环境需要大约1GB空间, 请在确保不需要的时候再删除旧引导. 可以使用系统→启动菜单创建和删除引导环境;

+ 建议使用品牌的U盘, ZFS系统可能会更容易在廉价的U盘中发生不可预计的错误;

+ 为了更加可靠的引导, 可以使用两个设备, 在安装时选择他们, 那么安装时会创建镜像引导设备.

####存储介质与控制器

FreeNAS®支持的存储控制器同FreeBSD, 可以在[列表](https://www.freebsd.org/releases/10.3R/hardware.html#disk "Disk Controllers Compatibility List")中搜索兼容的硬件设备.

FreeNAS®支持热插拔设备, 使用此功能需要在BIOS中启用AHCI.

使用HBA卡如Avago MegaRAID controller或者3Ware twa-compatible controller可以获得可靠磁盘劲爆以及实时的报告.

建议在磁盘添加到阵列前先做测试, 有可能的话坏块扫描也应当完成.

对于ZFS, [ZFS存储池对磁盘空间的要求](http://docs.oracle.com/cd/E19253-01/819-5461/6n7ht6r12/index.html "Disk Space Requirements for ZFS Storage Pools")推荐最少有16GB的磁盘空间. 由于ZFS会创建swap(交换)分区, 因此小于3GB的磁盘是不可以使用ZFS的. 但是, 当磁盘空间低于推荐值时, 将失去较大的空间用于建立swap, 例如在一个4GB的磁盘中, 2GB空间被用作swap.

想要为ZFS购买硬件的新手建议阅读[ZFS存储池建议](http://www.solarisinternals.com/wiki/index.php/ZFS_Best_Practices_Guide#ZFS_Storage_Pools_Recommendations "ZFS Storage Pools Recommendations").

ZFS虚拟磁盘设备vdevs, 是作为单个设备使用的磁盘组, 它可以使用不同大小的磁盘创建, 但是每个磁盘的可用空间会被限制为磁盘组中容量最小的磁盘空间大小. 例如, 2TB和4TB的硬盘组成的vdev设备每一个磁盘都最多只能用2TB空间, 通常情况下使用大小相同的磁盘来获得最佳的使用空间以及性能.

[ZFS设备大小与考校比较表格](https://forums.freenas.org/index.php?threads/zfs-drive-size-and-cost-comparison-spreadsheet.38092/ "ZFS Drive Size and Cost Comparison spreadsheet")可以用来比较不同数量的磁盘可用空间的大小.

####网络接口

FreeBSD支持的以太网设备列表中列出了所有支持的NIC, 其中表现最好的是Intel Chelsio, 有需求的情况可以单独购买这些NIC, Realtek网卡通常情况下表现不佳, 因为它并不提供独立的处理器.

最少建议使用一个千兆的网络接口, 在家庭使用环境中单个千兆网卡在使用现代的硬盘(SATA 3硬盘)时能够很轻松达到110MB/s, 为了获得更高的速度可以使用LACP绑定网卡(链路聚合), 当然这个功能还需要有一个支持此功能的网管交换机.

目前来说FreeNAS®并不支持InfiniBand, FibreChannel over Ethernet, wireless这几种类型

硬件的性能和分享方式都会影响网络性能的表现. 在同样的硬件条件下SMB的速度比FTP或者NFS更慢, 因为Samba是一个单线程的方案, 因此更快的CPU能够提升SMB的表现.

LAN远程唤醒(WOL)的支持由FreeBSD驱动决定. 

###安装与升级

####获取FreeNAS

最新版本的FreeNAS®可以在[官方网站](http://download.freenas.org/ "FreeNAS.org")下载.

+ .iso: 这个是带有启动功能的安装镜像, 它可以刻录到CD也可以放在U盘中使用;

+ .GUI_Upgrade.txz: 这个压缩包是固件更新的镜像. 如何更新FreeNAS®, 下载此文件, 阅读[升级].

每一个文件都有它相关的`sha256.txt`文件, 它可以用来确定下载的文件是否完整.

####准备安装介质

FreeNAS®的安装介质可以是CD或者U盘.

`.iso`文件既可以用于CD的烧录也可以用于U盘的刷写. 安装系统之前, 需要将BIOS的启动项修改为FreeNAS®安装介质.

+ 在 FreeBSD 或者 Linux 平台

	使用 dd 写入

```
dd if=FreeNAS-9.10-RELEASE-x64.iso of=/dev/da0 bs=64k
6117+0 records in
6117+0 records out
400883712 bytes transferred in 88.706398 secs (4519220 bytes/sec)
```

+ 在 Windows 平台

	使用[Image Writer](https://launchpad.net/win32-image-writer/ "Image Writer")或者[Rufus](http://rufus.akeo.ie/ "Rufus")写入安装镜像
	
####安装

1.将安装介质插入电脑, 并且从该介质启动,  FreeNAS® 安装 GRUB 菜单如图所示.

*注: 如果没有正确启动, 那么应该进入BIOS, 修改首选项为CD或者USB设备启动*

![InstallGrub](http://doc.freenas.org/9.10/_images/install1.png "InstallGrub")

2.等待GRUB菜单超时或者按下`Enter`进入安装界面.

![WelcomeMenu](http://doc.freenas.org/9.10/_images/install2c.png "Welcome")

3.按下`Enter`选择默认的`1 Install/Upgrade`进入安装或者升级界面, 在下一个菜单中, 安装向导列出了所有可用的目标介质, 包含USB设备, 他们的标识从__da__开始.

![Listdevice](http://doc.freenas.org/9.10/_images/install3a.png "Listdevice")

4.使用方向键将高亮光标移到目标USB设备, SSD, DOM, 内嵌闪存, 或者虚拟磁盘中. 按下`Spacebar`选中它, 被选中的项目会显示`[*]`, 此处如果选中第二个设备, 则创建镜像启动设备, 选择完成后, 将光标移动到OK上(或者按`O`), 然后按下`Enter`. 其他未选中的磁盘则作为存储设备来使用. 系统会提示Warning, 使光标停留在`Yes`上, 按下`Enter`.

![Selectdevice](http://doc.freenas.org/9.10/_images/cdrom3a.png "Selectdevice")

5.安装程序会确认在选择的设备中是否含有FreeNAS® 8.x 或者 9.x 系统, 如果存在, 则显示下方的提示, 如果要全新安装, 则应当选择`Fresh Install`并且按下`Enter`进行全新安装.

![Freshinstall](http://doc.freenas.org/9.10/_images/upgrade1a.png "Freshinstall")

6.系统会出现如图所示菜单, 提示你设定*root 密码*, 此处密码可以用于登录WebGUI. 输入两次相同的密码按下`Enter`则完成密码设定.

![Setpasswd](http://doc.freenas.org/9.10/_images/install4a.png "Setpasswd")

7.FreeNAS®可以设置为通过BIOS或者EFI启动, 对于老旧设备一般推荐BIOS启动方式, 而新设备使用UEFI启动方案.

![BIOSorUEFI](http://doc.freenas.org/9.10/_images/install5.png "BIOSorUEFI")

8.等待安装进程结束, 则出现以下界面. 按下`Enter`回到安装界面, 选择完成后`3 Reboot System`并且移除安装介质, 则系统将会稍后通过安装好的引导启动.

![Success](http://doc.freenas.org/9.10/_images/cdrom4a.png "Success")

####升级
目前来说FreeNAS®的升级主要是两种方案

+ 通过ISO升级

	大体方法与安装相同, 但是在此页面选择`Upgrade Install`
	
![ISOUpgrade](http://doc.freenas.org/9.10/_images/upgrade1a.png "ISOUpgrade")

+ 通过WebGUI升级

	在界面中选择`System → Update`即可完成升级
	
###启动

控制台的初始化菜单如下, 在启动完成后就会显示, 当FreeNAS®拥有显示器键盘等外设的时候, 这个控制台就可以作为系统的Administer控制系统.

*注: 控制台菜单可以通过FreeNAS® GUI 通过在Shell中输入指令__`/etc/netcli`__. 控制台菜单可以通过`System → Settings → Advanced → Enable Console Menu`开启或者关闭*

![ConsoleMenu](http://doc.freenas.org/9.10/_images/console1b.png "ConsoleMenu")

这个菜单提供了这些功能:

1)配置网络接口: 提供了一个设定系统网络接口的向导;

2)配置链路聚合: 允许建立一个新的链路聚合或者删除已经存在的链路聚合;

3)配置VLAN接口: 可以被用来设定VLAN

4)设置默认路由: 设置IPv4/IPv6的默认网关

5)设置静态路由: 按照提示设定目标网络以及网关IP, 每一条路由都需要再次输入此选项

6)设置DNS: 当提示的时候设置DNS域名以及首选DNS服务器的IP地址. 当添加多个DNS服务器的时候, 按下`Enter`来添加下一个, 按两次`Enter`则退出此选项

7)设置Root密码: 如果你不能登录图形界面, 选择此项可以重新设定root密码

8)重置出厂: 删除所有在界面中的配置更改, 选择此项以后, 系统将会重启, 如果有需求, 可以通过`Storage → Volumes → Import Volume `重新导入卷

9)Shell: 开启FreeBSD的命令终端, 使用exit退出终端

10)System Update: 检查系统更新, 如果有可用的更新, 它会被自动下载并应用. 这是WebUI上Update的简化版, 更新是被立即应用的, 无须通过GUI确认, 其他高级选项需要通过WebUI中的Update

11)创建卷备份: 创建FreeNAS® 系统配置备份以及ZFS设定, 同时可以选择通过加密链接备份数据到一个远程的系统

12)从备份中重载卷: 从选项11或者WebUI中`System → Advanced → Backup`恢复已经存在的备份文件.

13)重启

14)关机

####获取一个IP

在启动完成之后, FreeNAS® 会自动尝试通过DHCP方式获取到一个动态的网络接口, 如果成功的到了地址, 那么这时候地址会被显示在终端控制台的界面上，

一些FreeNAS® 是建立在没有显示器的环境中的, 那么获取IP地址将会是困难的, 如果这个网络能够支持Muticast DNS(mDNS), 那么可以通过访问主机名来得到ip, 默认的值是*freenas.local*

如果FreeNAS® 服务器没有成功的获取到IP, 那么还可以手动通过控制台为它分配IP地址: 比如说这台FreeNAS系统有一个网络接口, 名称为*em0*

```
Enter an option from 1-14: 1
1) em0
Select an interface (q to quit): 1
Reset network configuration (y/n) n
Configure interface for DHCP? (y/n) n
Configure IPv4? (y/n) y
Interface name: (press enter as can be blank)
Several input formats are supported
Example 1 CIDR Notation: 192.168.1.1/24
Example 2 IP and Netmask separate:
IP: 192.168.1.1
Netmask: 255.255.255.0, or /24 or 24
IPv4 Address: 192.168.1.108/24
Saving interface configuration: Ok
Configure IPv6? (y/n) n
Restarting network: ok
You may try the following URLs to access the web user interface:
http://192.168.1.108
```
在得到FreeNAS® 系统的IP以后, 在浏览器中输入此IP就可以进入FreeNAS® 的WebUI控制界面.

####登录
使用安装过程中设定的root密码即可登录GUI界面

![Login](http://doc.freenas.org/9.10/_images/login1b.png "Login")

![Loginmain](http://doc.freenas.org/9.10/_images/initial1c.png "Loginmain")

####登录
使用安装过程中设定的root密码即可登录GUI界面

![Login](http://doc.freenas.org/9.10/_images/login1b.png "Login")

![Loginmain](http://doc.freenas.org/9.10/_images/initial1c.png "Loginmain")

###账户

账户配置即通过GUI界面手动创建、管理账户和用户组, 此部分包含以下条目: 

+ 用户组: 管理FreeNAS®系统中UNIX风格的用户组

+ 账户: 管理FreeNAS®系统中UNIX风格的账户

####配置用户组

用户组界面提供了FreeNAS®系统中UNIX风格的用户组的管理功能.

*注意: 如果目录服务(directory service)运行于网络中, 那么就没有必要重新创建一个网络用户或者用户组. 同时, 为FreeNAS®导入已经存在的账户信息是参考目录服务的*

这个选项的内容是创建用户组并且为用户组分配账户的方法.

点击`Groups → View Groups`可以看到如图所示的信息:

![Listgroup](http://doc.freenas.org/9.10/_images/group1.png "Listgroup")

这里列出了系统中所有的用户组, 每一个用户组都有它的组ID, 组名, 它是不是一个FreeNAS®内建的组, 这个组成员是不是被允许使用sudo. 点击一个组, 下方会出现`Member`按钮, 点击这个按钮就可以改变这个组的成员.

点击`Add Group`, 则显示如图所示的创建组对话框

![addgroup](http://doc.freenas.org/9.10/_images/group2.png "addgroup")

创建一个新的组:

|设定		|值			|描述
|:----------|:----------|:----------
|用户组 ID	|字符串		|自动填充一个可用的ID; 按照约定, 含有帐户的UNIX用户组ID应该大于1000, 用户组的ID会等于默认的服务端口号(例如sshd用户组的ID是22)
|用户组名称	|字符串		|强制
|许可Sudo	|复选框		|如果复选框被选中, 组内成员有使用sudo的权利; 当使用sudo的时候, 用户会被提示输入自己的密码
|允许重复GID|复选框		|允许多个组分享相同的组ID(GID);当GID与现有数据的UNIX权限有关联的时候, 将会非常有用

当用户组和用户被创建后, 用户可以成为一个组的成员. 选中成员所分配的用户组, 点击`Members`选中`Member users`列表(这里会显示系统中所有用户账户)单击`>>`来移动用户到右侧. 显示在右侧的用户帐户会被添加成为当前用户组的成员

####配置账户

FreeNAS®支持账户, 用户组, 权限, 同时可以自由配置没个账户可以访问的FreeNAS®中的数据. 为了分配分享权限, 必须完成下列步骤:

1.创建一个所有用户都可以使用的访客账户, 或者为每一个网络上的用户设置与电脑登录账户相同的账户. 举例来说: 比如Windows系统有一个账户名称为*bobsmith*, 那么在FreeNAS®系统中也创建一个*bobsmith*账户. 一般情况下, 创建带有不同权限的用户组, 并将账户分配到这些用户组中.

2.如果你的网络用户有一个目录服务(directory service), 可以将这些已经存在的账户信息导入.

`Account → Users → View Users`提供了在FreeNAS®中用户的列表.

![Listaccount](http://doc.freenas.org/9.10/_images/user1a.png "Listaccount")

每一个账户都有账户ID, 用户名, 主用户组ID, 主目录, 默认Shell, 全称, 是否内建账户, E-mail, 是否禁止登录, 账户是否被锁定, 是否可以使用sudo提权, 账户是否与Windows 8以及更高版本的系统连接. 如果需要重新排序, 单击所需列名称, 则按照这一列进行排序, 再次点击则逆序输出.

点击一个用户账户, 则出现以下两个按钮:

+ 修改用户(Modify User): 用来修改账户设置;

+ 修改E-mail: 用来修改账户关联的E-mail地址.

*注意: 为内置的root账户设置email地址是非常重要的, 因为重要的系统信息会被发送到该地址. 出于安全考虑, 应当禁止通过密码登录root账户, 同时这个规则最好不要被修改.*

除了root账户, 其他FreeNAS®内建账户都是系统账户, 每一个系统账户都是用于服务的, 不应该用作登陆帐户. 因此, 默认的shell账户是nologin账户. 出于安全考虑, 为了不破坏系统服务, 请不要修改系统账户.

`Add User`按钮打开后如图所示. 一些设置仅出现在`Advanced Mode`. 为了看到这些设置, 需要点击`Advanced Mode`, 或者将系统设置为默认显示高级设定(`System → Advanced`中的`Show advanced fields by default`复选框). 

![Addaccount](http://doc.freenas.org/9.10/_images/user2.png "Addaccount")

|设定				|值					|描述
|:------------------|:------------------|:------------------
|账户ID				|整型				|如果已经被使用过将显示灰色; 当创建账户的时候, 下一个ID的数字会被自动显示; 按照约定, 用户账户的ID应当大于1000, 同时系统账户拥有与服务端口号相同的ID
|用户名				|字符串				|如果已经被使用过将显示灰色; 最多16个字符, 推荐不要超过8个字符; 不可以连字符开头, $只可以是为ui后一个字符, 同时用户名不可包含空格, 制表符以及, : + & # % ^ & ( ) ! @ ~ * ? < > = “
|创建一个新的首选组	|复选框				|默认情况下, 创建于账户名称相同的用户组; 不选中此复选框, 则可以使用与用户名不同的用户组
|主用户组			|下拉菜单			|必须不选中`Create a new primary group`才可以启用这个菜单; 为了安全原因, FreeBSD不会为主用户组为wheel的账户分配sudo权限; 如果需要添加su权限, 需要在`Auxiliary groups`(辅用户组)中添加*wheel*用户组
|主目录				|浏览按钮			|浏览一个已经存在的卷或者数据集, 此目录会被分配给该账户
|主目录模式			|复选框				|仅在`Advanced Mode`出现, 同时内建用户的该选项只读; 设置用户主目录的默认Unix权限
|Shell				|下拉菜单			|为本地登录和SSH登录选择Shell的种类
|全称				|字符串				|强制, 可以包含空格
|电子邮件			|字符串				|账户相关联的电子邮件地址
|密码				|字符串				|强制, 除非选择了`Disable password login`; 同时不可以包含`?`
|确认密码			|字符串				|必须与密码相同
|禁止密码登录		|复选框				|选中此复选框, 则禁止使用密码登录以及通过SMB共享的验证; 不选中时, 需要为用户设定一个密码; 选中此复选框时, 锁定用户以及允许使用Sudo均不可用
|锁定用户			|复选框				|在锁定用户时, 用户不可登录; 选中此选项时会禁用使用密码登录
|允许使用Sudo		|复选框				|如果选中, 那么这个用户组的成员就有使用sudo的权限; 当使用sudo的时候, 应当输入每个账户自己的密码
|Microsoft账户		|复选框				|选中此复选框, 链接到Windows 8或者更高版本的系统
|SSH公钥			|字符串				|粘贴此用户的SSH授权公钥(请不要输入私钥)
|辅用户组			|鼠标选择			|点击使得你想添加的用户组名称高亮, 然后使用>>按钮添加用户组

*注意: 有一些项目内建账户不可修改*

|Shell				|描述
|:------------------|:------------------
|netcli.sh			|用户可以使用控制台的初始化菜单, 即使在`System → Advanced → Enable Console Menu`被禁用的状态
|csh				|C shell
|sh					|Bourne shell
|tcsh				|增强的 C shell
|nologin			|在创建系统账户或者创建可以使用共享身份验证但是不可以登录到FreeNAS®系统的账户时使用
|bash				|Bourne Again shell
|ksh93				|Korn shell
|mksh				|mirBSD Korn shell
|rbash				|受限的 bash
|rzsh				|受限的 zsh
|scponly			|选择scponly使得SSH仅用作scp和sftp命令
|zsh				|Z shell
|git-shell			|受限的 git shell

###系统

管理账户的WebUI界面中"系统"部分包含以下条目:

+ 信息: 提供了FreeNAS®中常规的系统信息, 例如主机名, 操作系统版本, 硬件平台, 在线时长

+ 常规: 可以配置常规设置, 比如HTTPS登录, 语言, 时区

+ 启动器: 可以创建, 重命名, 以及删除启动环境

+ 高级: 可以配置高级设置, 比如串口控制台, 交换区, 串口控制台信息等

+ 电子邮件: 可以配置接受提醒的电子邮件地址

+ 系统数据集: 可以配置本地的日志和报告存储位置

+ 微调: 提供了一个实时调优的前端以及在引导时可以加载额外的内核模块

+ 更新: 可以执行系统更新以及检查系统更新

+ CA机构: 可以导入或者创建内部或者中级CA机构

+ 证书: 可以导入已经存在的证书或者创建自签证书

+ 支持: 可以用来报告Bug或者请求一个新的功能

####信息

`System → Information`显示了FreeNAS®系统的常规信息.

这些信息包含主机名, 版本, CPU, 内存, 系统时间, 在线时长, 平均负载.

点击`Edit`按钮可以修改系统的主机名, 修改完成后点击`OK`. 主机名必须包含域名, 如果网络不使用域, 那么在主机名的结尾加上*.local*

![Information](http://doc.freenas.org/9.10/_images/system1d.png "Information")

####常规

`System → General`如图所示

![General](http://doc.freenas.org/9.10/_images/system2b.png "General")

|设定				|值					|描述
|:------------------|:------------------|:------------------
|协议				|下拉菜单			|选择连接到管理WebUI的协议; 如果从默认的*HTTP*修改为*HTTPS*或者*HTTP+HTTPS*, 选择证书选项来使用证书; 如果没有证书, 那么需要首先创建一个CA, 然后创建证书
|证书				|下拉菜单			|*HTTPS*协议需要该证书; 可以选择本地的证书来加密链接
|WebGUI IPv4地址	|下拉菜单			|从最近的IP地址中选择, 限制可以使用管理GUI界面的IP地址; 内建的HTTP服务器自动绑定到通配符地址0.0.0.0(任意IP地址), 如果指定的IP地址不可用, 会出现警告
|WebGUI IPv6地址	|下拉菜单			|从最近的IP地址中选择, 限制可以使用管理GUI界面的IPv6地址;内建的HTTP服务器自动绑定到任意地址, 如果指定的IP地址不可用, 会出现警告
|WebGUI HTTP端口	|整型				|允许配置通过非标准端口访问HTTP管理界面; 更改此设置可能需要同时修改浏览器的默认配置
|WebGUI HTTPS端口	|整型				|允许配置通过非标准端口访问HTTPS管理界面
|HTTP→HTTPS重定向	|复选框				|当这个复选框被选中, 同时选择了HTTPS协议时, *HTTP*的链接会被自动重定向到*HTTPS*
|语言				|下拉菜单			|从下拉菜单中选择本地化语言, 并且重新加载浏览器
|控制台键盘映射		|下拉菜单			|选择键盘映射
|时区				|下拉菜单			|从下拉菜单中选择时区
|系统日志等级		|下拉菜单			|如果定义了`Syslog server`, 只发送符合这个等级的日志
|系统日志服务器		|字符串				|远程系统日志服务器的地址: `[IP地址或者主机名]:[可选端口数字]` 设置后, 日志会被写入控制台和远程服务器

在做出任何改变以后都需要点击`Save`按钮可以修改系统的主机名

这个界面中同时有以下按钮:

重置出厂设置: 将配置数据重置到该版本默认的设置. 但是它不会删除用户的SSH密钥或者用户在自己目录中存储的任何数据. 所有已经存储的配置变化都会被清除, 这个选项是为了在出现配置问题是可以尝试重置系统到原始配置.

保存设置: 将当前配置数据库的备份副本以hostname-version-architecture格式保存到用于访问管理界面的系统. 建议在进行任何配置更改后始终保存配置. FreeNAS®每天早晨的3:45都会自动将配置数据库备份到系统数据集, 但是如果FreeNAS®在此期间处于关闭状态, 则无法进行备份. 如果系统数据被存储在引导池中, 同时引导池不可用, 那么备份也将不可用. 可以使用`System → System Dataset`查看或者修改系统数据库的位置.

*警告: 密码会在系统配置中备份. 有两种类型的密码, 一种是操作系统的用户帐户密码, 它们会被存储为hash值, 不需要使用加密保证安全, 同时会被保存在系统配置备份中. 其他的密码(例如iSCSI的CHAP密码或者Active Directory绑定的凭据)必须以加密的形式存储, 防止他们在已经保存的系统设置中以明文显示. 这些加密密匙都被保存在引导设备上, 如果FreeNAS®被安装到一个新的引导设备上, 同时系统配置备份被移动到这个新的引导设备中, 他们的密匙将不会存在, 需要被重新输入.*

上传配置: 允许通过浏览本地先前保存的配置文件恢复其配置方案, 屏幕变为红色，表示系统将需要重新启动以加载恢复的配置.

NTP服务器: 网络授时协议(NTP)可以用作在网络上同步计算机的时间. 对于一些对时间敏感的服务(例如Active Directory或者其他目录服务)来说很重要. 默认情况下，FreeNAS®已预先配置了三个公共NTP服务器. 如果在网络中使用目录服务，请确保FreeNAS®系统和运行目录服务的服务器已配置为使用相同的NTP服务器. 如果要在FreeNAS®系统添加NTP服务器, 那么单击`NTP Servers → Add NTP Server`打开如图所示的表格.

![NTP](http://doc.freenas.org/9.10/_images/ntp1.png "NTP")

|设定				|值					|描述
|:------------------|:------------------|:------------------
|地址				|字符串				|NTP服务器的地址
|突发传输Burst		|复选框				|建议当`Max. Poll`超过10的时候使用; 仅使用在自建服务器上, 不要用在公共NTP服务器上
|IBurst				|复选框				|加速初始化同步(以秒为单位)
|首选				|复选框				|仅当已知这个NTP服务器是高度精准的, 比如说那些有着时间具有时间监控硬件的
|最小 Poll			|整型				|2秒内的功率; 不可以低于4或者高于`Max. Poll`
|最大 Poll			|整型				|2秒内的功率; 不可以高于17或者低于`Min. Poll`
|强制				|复选框				|强制使用附加的NTP服务器, 即使目前不可用

####启动器

FreeNAS®支持被称为多引导环境的ZFS功能. 多引导环境使得系统升级变得更加低风险. 在应用升级之前升级程序会自动创建一个当前引导环境的快照, 同时把它添加到引导菜单. 如果升级失败, 重启系统并且从引导菜单中选择先前的启动环境, 以指示系统回到之前的系统状态.

*注意: 引导环境是与配置数据库分离的, 引导环境是操作系统在指定时间下的快照, 当FreeNAS®系统启动时, 它加载指定的引导环境， 或者操作系统, 然后读取配置数据库, 以此加载当前的系统配置. 如果想要进行配置更改而不是操作系统更改, 使用`System → General → Save Config`备份配置数据库*

就像下图所示, 在安装FreeNAS®的时候会创建两个引导环境. 系统会引导默认的引导环境, 用户可以自己改变或者从这个版本升级, 其他的引导环境被称为*Initial-Install*(初始安装), 当系统需要回到未配置的版本, 则可以选择此项引导.

如果使用向导, 会创建第三个称为*Wizard-date*的引导环境, 包含向导运行时的日期和时间.

![bootconfig](http://doc.freenas.org/9.10/_images/be1g.png "bootconfig")

每一个引导环境条目都包含以下信息:

+ 名称: 引导条目在引导菜单中显示的名称

+ 活动: 显示在用户不做出选择时默认引导的引导条目

+ 已创建: 显示引导被创建的日期和时间

+ 保持: 显示了如果更新没有足够的空间, 此引导环境是否可以被删除. 如果该引导环境不应该被自动删除, 则启用此处的Keep属性

点击使得一个条目高亮, 则显示以下的按钮:

+ 重命名: 用来改变引导条目在引导菜单中显示的名称

+ 保持/不保持: 用于切换更新程序是否可以在没有足够空间继续更新的时候裁剪(自动删除)此引导环境是否可以被删除

+ 复制: 用来创建高亮引导环境的拷贝

+ 删除: 用来删除高亮的条目, 同时可以从启动菜单中移除条目. 你不可以删除一个处于活动状态的条目, 这个按钮将不会显示在活动状态的引导环境下. 如果你需要删除目前活动的条目, 首先激活另一个条目, 这就会清除当前条目的活动状态. 注意, 默认引导环境不会显示此按钮, 因为只有这个条目才可以将系统恢复到出事安装状态

+ 活动: 只出现在没有被设置为活动状态的条目下, 改变选中的条目使得该条目作为下一次启动的默认条目. 它的状态会被改变为`On Reboot`(在重启时), 而先前的启动项从`On Reboot, Now`(在重启时, 现在)变为`Now`(现在), 表示先前的启动项在下一此引导时不会被默认启用

引导条目上方的按钮有以下功能:

+ 创建: 手动创建引导环境, 弹出菜单会提示输入引导环境的名称, 输入名称是只允许使用字母, 数字, 字符, 下划线, 破折号

+ 启动器垃圾清理: 可以会引导设备手动清理, 通常情况下, 引导设备每35天会自动清理一些, 如果要更改默认间隔, 请在自动清理周期字段中填入数字. 最后一次清理的日期和结果也会被列在此页面中, 启动卷状态应当显示`HEALTHY`

+ 状态: 点击这个按钮可以看到引导设备的状态, 引导只有一个启动设备处于ONLONE状态.

![bootstatus](http://doc.freenas.org/9.10/_images/be2.png "bootstatus")

如果系统存在镜像引导设备, 那么有一个引导设备的状态会显示OFFLINE, 点击设备, 再点击`替换`俺就可以重建引导镜像.

请注意, 如果引导设备是唯一的引导设备, 则它不能替换引导设备, 因为它包含操作系统.

![bootmenu](http://doc.freenas.org/9.10/_images/be3c.png "bootmenu")

第一个条目就是活动的引导环境, 或者是引导已经配置的系统. 如果要引导其他的引导环境, 那么按下`spacebar`暂停在引导菜单, 使用方向键选择`Boot Environment Menu`, 按下`Enter`, 则显示其他可用的引导环境. 使用上下按键选择引导环境, 按下`Enter`继续引导, 如果每次都需要使用该引导, 则应当进入`System → Boot`, 激活该启动项.

#####镜像启动设备

如果系统当前从一个设备引导, 你可以添加另外一个镜像启动设备. 通过这种方式, 当一个设备启动失败时, 系统仍然有一个备份的启动文件系统, 因此可以配置从剩余的镜像设备中启动.

*注意: 当添加另外一个引导设备的时候, 它必须与已存在的设备大小相同(或者更大). 不同类型的USB设备可能不具有相同的大小, 因此建议使用相同型号的USB设备.*

在下图中, 用户单击`System → Boot → Status`, 将当前的启动状态显示在页面中. 在图中, 当前的设备是*ada0p2*, 它目前处于*ONLINE*状态, 同时它现在是唯一的一个引导设备. 点击任意一个叫做*freenas-boot*或者*stripe*条目点击附加按钮, 如果有另一个可用设备, 他会显示在成员盘下拉菜单中.选择目标设备, 点击添加.

![addmirror](http://doc.freenas.org/9.10/_images/mirror1.png "addmirror")

一旦镜像设备创建完成, 状态页面中会显示现在是一个*mirror*, 设备会显示在mirror中.

####高级

`System → Advanced`菜单选项如下所示.

![advancemenu](http://doc.freenas.org/9.10/_images/system3b.png "advancemenu")

|设定						|值					|描述
|:--------------------------|:------------------|:------------------
|启用控制台菜单				|复选框				|不选中则禁用控制台菜单
|启用串口控制台				|复选框				|不选中则禁用串口输出
|串口地址					|字符串				|串口的16进制地址
|串口速率					|下拉菜单			|选择串口使用的速率
|启用屏幕节能				|复选框				|启用/禁用控制台屏幕节能
|启用节能(节电后台程序)		|复选框				|使用[powerd](https://www.freebsd.org/cgi/man.cgi?query=powerd "powerd")监控系统状态并且设置CPU频率
|交换区尺寸					|非0整数(GB)		|通常情况下, 所有的数据盘会创建这个大小的交换区; 此设置不会影响日志或缓存设备, 因为他们实在没有交换区的情况下建立的
|在页脚中显示控制台信息		|复选框				|在浏览器的底部时时显示控制台信息; 单击控制台以显示可滚动屏幕; 勾选`Stop refresh`复选框可以在滚动屏幕中暂停更新信息, 不勾选则继续显示新的信息
|出现致命错误时显示跟踪		|复选框				|发生致命错误时提供诊断信息的弹出窗口
|默认显示高级参数			|复选框				|一些界面提供了`Advanced Mode`作为附加选项; 启用此选项则默认显示这些功能
|启用自动协调				|复选框				|启用[自动协调](#自动协调),使用这个可以根据硬件尝试优化系统
|启用debug版本内核			|复选框				|选中时, 启用debug版本的内核
|启用自动上传内核奔溃转储	|复选框				|选中时, 内核的崩溃转储(系统的状态, 收集到的RRDs, 以及选择一些syslog信息)自动上传给开发团队
|MOTD 横条					|字符串				|用户登录SSH时显示的信息
|定期通知用户				|下拉菜单			|用户通过email收到安全报告; 这个输出每夜都会运行, 但是只有系统重启或者遇到错误时才会发送邮件
|远程Graphite服务器主机名	|字符串				|运行Graphite服务的远程服务器IP地址或者主机名
|日志启用FQDN				|复选框				|选中时, 日志包含有完全限定的域名以此精确标识有相似主机名的系统

在做任意更改后需要点击`Save`按钮.

这个菜单中也包含这些按钮:

备份: 通过加密链接备份FreeNAS®的设置, ZFS的设定, 同时可以选择备份数据到远程系统中. 点击按钮可以打开如图所示的配置页面. 远程系统需要有足够的空间, 并且有运行在22端口的SSH服务, 由于备份是二进制文件,远程系统不一定要格式化为ZFS, 可以使用控制台菜单中的`12) Restore from a backup`恢复配置.

*警告: 备份和还原选项适用于灾难恢复, 如果恢复系统, 则系统会被重置到建立备份的时候. 如果选择了备份数据选项, 那么备份之后创建的所有数据都会被清空. 如果没有选择备份数据, 那么只会恢复ZFS的配置, 不会恢复数据.*

*警告: 备份程序`IGNORES ENCRYPTED POOLS`(忽略加密的存储池), 不要使用此功能备份加密的存储池.*

保存Debug信息: 用来创建诊断信息的文本文件, 创建Debug文件时, 会提示生成的ASCII文本文件的位置.

![backmenu](http://doc.freenas.org/9.10/_images/backup1.png "backmenu")

|设定				|值					|描述
|:------------------|:------------------|:------------------
|主机名或者IP地址	|字符串				|输入远程系统的IP地址或者主机名(如果已经设定DNS)
|用户名				|字符串				|用户必须存在于远程系统中, 同时必须有写入`Remote directory`的权限
|密码				|字符串				|输入并确认账户的密码
|远程目录			|字符串				|保存备份的完整目录路径
|备份数据			|复选框				|默认情况下, 备份只是快速的备份配置数据库以及ZFS存储池的数据库; 选择这个复选框则同时保存数据(这个选项会需要一些时间, 时间由备份的存储池大小以及网络的速度决定)
|压缩备份			|复选框				|选中时, 使用gzip压缩备份数据来减少传输数据的尺寸
|使用授权密匙		|复选框				|选中时, root账户的公钥必须被存储在远程系统的`~root/.ssh/authorized_keys`中, 同时这个密钥不应当受到密码保护

#####自动协调

FreeNAS®提供了一个自动协调脚本来尝试根据硬件优化系统设置. 比如说, 如果系统中存在一个ZFS卷有着RAM限制, 这个自动协调脚本会自适应一些ZFS sysctl来减少ZFS对内存的需求问题. 它应当作为一个临时措施, 只到系统添加更多的内存. 由于自动协调限制了ARC, 因此它会降低系统速度.

自动协调默认是禁用状态, 勾选后使得自动协调脚本在启动时运行, 如果希望系统立马执行脚本, 那么系统必须重启.

如果自动协调脚本找到一些可以优化的设置, 变化会显示在`System → Tunables`， 如果不喜欢这个设置, 可以编辑在GUI界面显示的值, 同时会覆盖自动协调脚本创建的值. 但是删除自动协调脚本创建的微调, 那么这项值会在下一次启动中重新创建.

如果你想要提升FreeNAS®系统的性能, 同时怀疑当前系统性能被限制, 可以尝试开启自动协调脚本.

如果你希望查看脚本执行了哪些检查, 可以在`/usr/local/bin/autotune`找到脚本.

####电子邮件

自动脚本每天都会向root账户发送包含磁盘健康等重要信息的邮件. 警告时间也会被发送到root账户. 但是, 管理账户通常不会在FreeNAS®系统中阅读邮件. 那么这些email通常会被额外发送到管理员方便阅读的电子邮箱地址. 通过配置这个可以是的管理员的远程邮箱更容易收到系统问题, 状态变化的报告.

第一部是设定远程细条绒的邮箱地址, 点击*root*使得账户高亮, 再点击修改电子邮件并输入电子邮箱地址.

额外的设定显示在`System → Email`中.

![emailmenu](http://doc.freenas.org/9.10/_images/system4b.png "emailmenu")

|设定				|值					|描述
|:------------------|:------------------|:------------------
|来源地址			|字符串				|在发件人中显示的邮箱地址; 可以在接收系统中设置辅助过滤
|外部邮件服务器		|字符串或者IP		|用来发送邮件的SMTP服务器的主机名或者地址
|链接的端口			|整型				|SMTP的端口号, 通常来说是25或者465(安全SMTP), 587
|TLS/SSL			|下拉菜单			|加密的形式; 选择不加密, SSL或者TLS加密
|使用SMTP授权		|复选框				|启用或者经用SMTP的授权; 选中时需要输入用户名密码
|用户名				|字符串				|必要时输入SMTP的授权用户名
|密码				|字符串				|必要时输入SMTP的授权密码

点击`Send Test Mail`确认设置的电子邮件设定可以正常工作. 如果测试邮件发送是被, 双击目标邮件地址来修改.
