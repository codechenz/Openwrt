## 记录一下折腾软路由入到坑。

### 0.本地环境和网络架构
运营商开通了动态公网IP。入户光猫是桥接模式。直连软路由，期待软路由做一些科学上网，DDNS，签到等工作。然后直连一个交换机（H3C的一个8口千兆交换机），把信号分到各个屋里，其中一根线走到无线路由器供家中无线设备连接。无线路由器是小米的AX3600，原生固件，设置成有线中继模式，也叫AP模式，只负责用来发射WiFi信号。
大概得网络拓扑类似这个图，网上找的图。
![277e30a1-f3d5-4279-a227-f76563afc0be](https://github.com/codechenz/Openwrt/assets/15170922/dcc106e2-daa1-4bac-9bb2-5d088883c57b)
光猫：天邑TEWA-600AGM  
软路由：J1900的一个小机器  
交换机：H3C 8口千兆  
无线路由器：小米AX3600

### 1.选择软路由系统

首先是选择固件，市面上多数的都是基于Openwrt再开发的系统，因为自己比较小白，也不太想过于折腾，选择了iStoreOS这个固件作为软路由的系统。这个系统是国内一家公司做的，比较适合新手小白，很多设置都是自动生成的，也有一些三方软件可以安装，当然了必不可少的科学上网软件，都是支持的，我用的是OpenClash。还有其他好几种热门，可以满足大多数人的需求。

### 2.下载固件
选定了固件之后，就是去下载固件，当前更新的都是比较快的，一个月都能有好几个新版本。我的硬件是Intel的J1900，我用最新版（22.03.5-2023082509）的安装系统的时候总是遇到一些问题。后来改用了（21.02.3, 2022092414）就可以安装成功了。如果你也遇到类似的问题，可以选择一个旧一些的版本先安装上，之后再通过iStoreOS自己的在线升级固件，应该也可以。旧版本的系统大体的功能也都在，如果新版本没有你恰好想要的功能，旧版也是可以用的。下载连接的话 https://www.koolcenter.com/fw?firmware=iStoreOS&tag=default&version=default&arch=default&brand=default&alia=default 根据你CPU的架构选择相应的固件，我选择的是X86_64，至于有无EFI这个，较新的机型都是支持EFI的，这个我理解是一种启动方式。

### 3.刷入固件
#### a.U盘
大家常用的模式，刷固件的软件，Mac的话使用balenaEtcher， win的话用rufus。然后找一个大于4G的U盘，插入电脑，选择好刚才下载的固件，刷入即可。

#### b.直接刷入目标硬盘（建议）
如果你有硬盘转USB的转接口的话，就可以用这个方式。例如Sata转USB，这样你就可以把要刷入的硬盘，连接到电脑上。然后使用磁盘管理工具格式化磁盘。Mac可以用自带的磁盘工具，Win的话可以使用DiskGenius。将磁盘格式化，删除多余的分区，然后只创建一个分区，格式的话选择exFAT4,没记错的话。这个应该是Linux系统所需的格式。NFTS是windows，AFPS是macos的。然后还是使用balenaEtcher或者rufus，像往U盘刷系统一下，把系统刷到格式化好的盘中。rufus可能需要将.gz格式的解压成img或者iso格式。刷完之后，你的硬盘就已经成功安装了iStoreOS系统，把硬盘安装回软路由，然后上电开机。直接跳到第5步。

### 4.安装系统
使用U盘的需要这一步，为你的软路由准备一个显示器和一个键盘。软路由开机，和安装windows系统一样，开机按Delete进入BIOS系统，修改boot启动为U盘启动。这里不同主板的话进入BIOS的按键可能不一样，可以google查一下，多数是F12 F10 Delete这些。然后系统会从U盘中启动，等待跑码。直到出现Openwrt字样或者iStoreOS字样。这里不清楚是不是固件的问题，有时候跑码会卡主，在20秒左右的时候，可以按一下回车就能出来Openwrt字样了。  

此时你可以和系统交互，输入quickstart，或者q自动补全。弹出的6个选项把。  
第一个是查看当前的interface，你可以通过这个看你的网口那个是WAN口那个是LAN口，我就被这里坑过，WAN口是需要连到光猫，LAN口连到交换机或者无线路由器。  
第二个change lan ip，这个是修改后台的ip的，默认是192.168.100.1, 如果你之前是192.168.1.1这里建议你还设置成192.168.1.1。因为如果你之前链接过的设备设置了静态IP，例如NAS，设置了192.168.1.8的话，由于是静态IP，NAS还无法登录后台修改设置重新加入192.168.100.1的号段，就会导致你的nas一直无法连接到新的ip号段。
第三个install to x86，这个就是安装程序，选择这个将系统镜像安装到硬盘上，注意这里不要选择错了硬盘，不要按到U盘上。不清楚是固件的问题还是什么，这里有时即使选择对了硬盘，还是有机会安装到U盘上或其他盘上。所以如果可以直接刷入硬盘，推荐用3b方法。
剩余两个一个是恢复出厂设置，这个应该是在你安装系统后，想为iStoreOS重置一些用户设置的时候用到，最后一个就是quit。  
选择install 安装系统，等待安装完成，提示拔掉u盘，重启设备。拔掉u盘，等待重启，boot启动会自动切换到系统盘。

### 5.登录iStoreOS系统
系统打开后，会出现和安装时候一样的openwrt或istoreos字样。此时你还可以做一些修改，例如改lan ip，进入工厂模式修改root密码等等。此时你可以连接局域网，或使用网线直连软路由。然后打开192.168.100.1，如果你没有更改lan ip的话，这个是默认的地址。root password是默认的用户名和密码。
