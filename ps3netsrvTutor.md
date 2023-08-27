# ps3如何通过网络运行ISO

## <center>前因
我的ps3一直是通过移动硬盘读取iso，加上ps3破解机制的问题，所以硬盘必须准备两个分区，一个FAT和一个NTFS。

FAT的兼容性是最好的，几乎所有的iso都可以通过FAT读取，但是由于FAT的4G大文件限制，有些游戏必须还要split然后再安装一部分数据到ps3内置硬盘，很繁琐。
而NTFS没有大小限制这一点是最棒的，只是有的iso从NTFS的分区加载后运行不了，不知道具体是啥原因，但这样的iso还是比较少的。

每次开机我的ps3都要挂上两个大硬盘（我没有单盘分两个区，而是用了两个不同格式的硬盘）, 加上我的ps2的iso读取一直是通过远程的方式([ps2通过smb加载iso教程](https://waizui.github.io/ps2guide/ps2guide.html))
所以我在考虑是不是可以让ps3也远程读取iso？这样就可以去掉外挂硬盘。

## <center>前期准备

* 一台破解了并且安装了multiman的ps3，如果未破解参考[webman-mod](https://github.com/aldostools/webMAN-MOD)
* 一台运行linux，windows，macOS中一个系统的电脑
* 去这个地址下载ps3netsrv程序 [releases](https://github.com/aldostools/webMAN-MOD/releases)

<b>需要注意的一件事，我在Win10上运行最新ps3netsrv无法成功，最后降级到[这个版本](https://github.com/aldostools/webMAN-MOD/releases/tag/1.47.23) 才成功，如果你使用最新版也无法成功，可以尝试降版本看看。</b>

## <center>设置步骤

ps3的LAN口支持千兆，为了发挥最大速度，需要买一根6类网线，加上一个支持千兆的路由器，如果只有百兆的路由器的话，iso读取速度最多到11MB/s 左右
速度很慢，有的游戏加载会花比较长的时间。

连接好ps3后我们记住ps3的ip地址，可以在你的路由器管理界面找到。

### Win10:
 首先建立存放ps3游戏的文件夹，比如我在D盘根目录建立PS3GAMES文件夹

 解压下载回来的ps3netsrv，找到windows的版本，进入文件夹后右键单击，然后选择在此处运行powershell，在弹出的界面中输入

    ./ps3netsrv.exe 路径参数 38008 ps3的ip

其中把路径参数和ps3的ip替换为实际数值，比如我的是

    ./ps3netsrv.exe D://PS3GGAMES/ 38008 192.168.0.103

然后回车，会出现运行成功的提示。这样我们就开启了ps3远程加载iso的服务端。

### Linux&macOS:
跟win10一样，建立专门存放的ps3游戏的文件夹

运行刚刚下载回来的ps3netsrv的对应版本，如果不能运行，记得给一下运行权限。打开终端，cd到对应的文件夹下，输入指令:

    sudo chmod +x ./ps3netsrv

完成后输入指令:

    ./ps3netsrv 游戏文件夹路径 38008 ps3的ip

其中把路径和ip替换为实际值。


### ps3:

打开ps3 进入multiman界面

在设置中找到到网络服务器这一项，确定后依次输入刚刚ps3netsrv运行的那台电脑的ip，端口(默认38008不用改)，以及名字(随便填)。完成后重新刷新游戏。

这时回到电脑上，会发现我们建立的ps3游戏文件夹下面多了一堆子文件夹，其中一个叫做PS3ISO，现在只要把iso放入这个文件夹。

然后回到ps3的multiman界面刷新游戏，应该就能看到右上角带有地球标记的游戏，那就是我们放在远程的游戏，剩下的就是像加载普通iso一样直接加载它即可。



## <center>注意事项:

* 有的iso加载后不能运行，这种情况只能在FAT硬盘里用普通加载方式运行了。


