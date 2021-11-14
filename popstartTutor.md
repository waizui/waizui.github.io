# 如何使用PopStarter在ps2上运行ps1游戏

## 前期准备 

<b>在这个地址可以找到完整的[英文教程](https://bitbucket.org/ShaolinAssassin/popstarter-documentation-stuff/wiki/quickstart-smb)
完整版含有smb，hdd和usb的教程，本教程只包含usb部分</b>

需要准备的材料有:

1. 一个已经通过FMCB破解的ps2，并且装得有wlaunchELF
2. 一个fat32的u盘
3. 去[这个仓库](https://github.com/AnimMouse/POPS-binaries)下载必须文件，全下回来,其中包含一个POPS_IOX.PAK文件，下一步骤会用到。
4. 在[这个地址](https://bitbucket.org/ShaolinAssassin/popstarter-documentation-stuff/downloads/POPStarter_USB_Quickstarter_Pack_20200209.zip)下载popstarter的运行文件和VCD文件的制作程序。
5. 一个ps1游戏文件，一般下载回来会包括一个bin和一个cue文件。

## 安装模拟器

6. 在u盘根目录新建一个叫做POPS的文件夹,把刚才下回来的POPS_IOX.PAK文件放到POPS文件夹下面。这个文件其实就是ps1的模拟器。

## 创建VCD文件

7. PopStarter唯一支持的游戏文件格式是.VCD格式，用.ISO和其他格式是运行不了的。所以我们需要从其他个格式转换为.VCD，方法
就是通过PopStarter提供的转换工具进行，这个工具在准备材料时的第四步中下载了,名字叫做CUE2POPS_2_3.EXE

8. 首先确保你下载的ps1游戏是由.bin文件和一个.cue文件组成的。然后将cue文件拖动到转换工具的图标上（也就是CUE2POPS_2_3.EXE）
然后就会在cue文件所在的文件夹下面生成与cue文件同名的.VCD文件。 然后把这个.VCD文件拷贝到刚刚U盘中新建的POPS文件夹下。

## 创建启动程序

9. 在4中下载的文件中找到POPSTARTER.ELF 然后把这个文件复制到上一步.VCD同级目录，然后把这个文件改名为成XX.游戏名字.ELF，比如游戏叫做
FF7.VCD 则改名为XX.FF7.ELF

10. 打开ps2 运行wlaunchELF，按圆圈键进行文件浏览，然后选中mass文件夹，这个就是我们盘根目录，进入前面建立的POPS文件夹
找到改名的POPSTARTER.ELF文件，比如上面说的XX.FF7.ELF然后圆圈键运行就可以了。


## 其他事项

* 如果游戏文件含有多个.bin文件的情况，需要先合并多个.bin不然没法使用CUE2POPS_2_3.EXE转换为.VCD文件。合并的方法是下载
CDmage这个程序,我下载的是1.02.1beta这个版本。然后运行程序，菜单栏找到File->Open 打开CUE文件,然后Save As 选一个新的文件夹保存，就能看到合并为一个的.bin文件。