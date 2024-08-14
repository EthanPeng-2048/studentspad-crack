# 这是我搬运的项目，这个项目不是我制作的，也没有我的贡献，它曾经在Github上发布过，但现在没了，这是原项目地址：https://github.com/sdgasdgahj/studentpad-research

# 关于'科大讯飞AI学习机(非课堂版机型)'的一些cr4cking and research

* 本项目欢迎各位读者开issue，**本项目秉承公益的原则，故禁止倒卖！被骗的欢迎开issue来提供骗子个人信息！**
* 如果看不了图片，你可以去下载WattToolkit，里面有“GitHub加速”功能；或者你也可以把这个项目克隆到本地，然后下个VSCode，VSCode中下一个OfficeViewer插件就能看了
* 材料：电脑*1，数据线一根，乐于提问、勤于思考/会用搜索引擎的人x1
* 文件库:https://kdxf.work/

# AI学习机高阶教程（Experimental）

* Warning：下列操作要想生效，都需要进行**解锁bl(即Bootloader锁)**，以及**刷入分区**的危险操作，三思而行，所以备份尤为重要，我建议是但凡是要进行修改的分区都应当进行备份，以及在root完成后，请使用爱玩机工具箱"导航——刷机工具箱——镜像分区管理"导出系统除userdata分区以外的所有分区的备份
* 下列操作都已经超出了《科大讯飞AI学习机AI学习软件服务用户协议》：

> 用户若对学习机进行刷机行为，包括但不限于获取root权限，刷入第三方ROM等，则本机将会从科大讯飞AI学习机官方技术支持和软件保修服务中被移除

所以，下面的内容请三思而行

## 获取学习机(展讯系机型)的root（即刷入Magisk）

为何要把root部分拿出来讲呢？因为展讯芯片系学习机的boot无法patch（原因是没有ramdisk）

system分区植入法目前已成功实践，rec法因为avb的原因无法正常启动，所以下文只讲system分区植入magisk法

适配机型:x2pro,t10等ud710设备，c系列机型（c6,c10,c10pro）因未知原因提取失败，

* 大概思路：将magisk安装到安卓system分区，可以参考[某酷安大佬写的方案（Magisk system root部分）](https://github.com/TomKing062/CVE-2022-38694_unlock_bootloader/wiki/Magisk)

总结下来就四步：①解锁板子的bootloader（这一步已经做到能纯Windows环境下进行了），②提取system分区并进行修改，③将修改好的文件放回分区并刷回板子中以root

### 解锁bootloader（最重要的一集）

* 文件：[到“紫光解BL锁”目录下下载adb-fastboot-win-unlock.zip文件](http://qutick102.ysepan.com/)
* 文件2：[下载&#34;用来得到解锁密钥的软件.apk&#34;](https://kdxf.work/)
* 进入fastboot模式(有两种方法，在各机型的新版本已经修复了第2种使用编程空间的方法,如失败请使用第1种方法进入fastboot):
* 1.使用物理按键进入REC（提示：与Jingpad A1进入rec的方式一致，具体方式请见Jingpad A1的Postmarket OS Wiki页面），然后轻按电源键+音量加的按键组合调出菜单，用音量上选择第二项，然后使用电源键进入fastboot模式
* 2.检查学习机内是否有编程空间软件，若没有可升级学习机系统；进入编程空间后输入以下文本并执行学习机会自动进入fastboot模式

```
import os
os.system("reboot bootloader")
```

#### 生成解锁密钥方式1（有另一台安卓设备的情况下）

* 在下载下来的文件的解压目录的地址栏输入cmd，在打开的窗口中输入 `fastboot devices `得到一串字母加数字，这就是你机器的序列号，在另一台安卓设备上安装“文件”中的[apk](https://www.alipan.com/s/SLgfA1dFGrt)，输入你得到的序列号（或平板背面贴纸上的SN码），完事会生成一个叫signature.bin的文件，你需要将这个文件拷到当前目录(即：你cmd在的目录)

#### 生成解锁密钥方式2(纯电脑方式)

* 步骤1.安装WSL环境(目的是为了得到Linux环境，你也可以利用VMware等虚拟机软件创建Linux环境，这里不做要求)
* 步骤2.在下载下来的文件的解压目录的地址栏输入cmd，在打开的窗口中输入 `fastboot oem get_identifier_token` 命令获取 android 设备ID

输出结果类似于这样：

```
Identifier token:
XXXXXXXXXXXXXXXXXXXXXXXX
OKAY [  0.019s]
finished. total time: 0.019s
```

* 将步骤 2 所获取的设备ID作为参数，在wsl中执行设备号签名脚本，命令如下：

  ```
  ./signidentifier.sh  <填入设备ID，若为多行，请将其连成一行，并且中间不能出现空格字符> rsa4096_vbmeta.pem signature.bin
  ```

取生成的signature.bin文件，这就是你需要的解锁文件

#### 正式解锁

* 使用 `fastboot flashing unlock_bootloader signature.bin`的命令，进行设备解锁，然后按下音量下键，确认解锁即可，然后bootloader会进行格机
* 恭喜你，成功解锁bootloader

### 进入download

关机，拔下数据线，然后，在维持 **长按音量下** 的情况下插上数据线即可进入下载

or:

关机，拔数据线，在维持 **同时按住音量下，电源键，音量上** 的情况下插入数据线即可进入下载

附赠：按键图

![1708767509011](image/README/1708767509011.png)

### 提取分区（重要：提取完请手动将两个镜像复制一份，防止出意外状况后无法恢复至原来状态）

#### 1.获取最新版本的spd_dump刷机工具

* 访问[qutick102.ysepan.com](http://qutick102.ysepan.com/)，这是刷机工具的存放网盘，由[@Tomking062](https://github.com/Tomking062)管理，打开之后，会出现如下界面：

  ![1718709531373](image/README/1718709531373.png)
* 你应该去下载最上面的**紫光驱动**以及最新版本的**spd_dump_dev**，你或许要问：那哪个是最新版本呢？很简单，找那个**后面跟着的日期最近的**就是，点一下文件名就能下载，另外要说明，你不应当**先安装紫光驱动**，这个驱动应当在你先把**bl正确解了以后**再安装(fastboot的驱动跟这个貌似有点冲突，会获取不到USB设备标识符导致识别不了)。
* 将紫光驱动解压，然后到**Driver_R4.21.3201\Doc**目录下，下面有一个安装教程，名为**Driver Install and Uninstall Guide V1.2 (CN).pdf**，照着这个自己安装驱动即可
* 将spd_dump_dev_日期.zip解压，将它里面的SPRD目录单独取出，放在一个你记得住的位置(for example:桌面)，其他的你删了都行。
* 打开那个留下来的SPRD目录，对空白处进行**Shift+右键** ，点击"在此处打开命令窗口"，输入**spd_dump_interactive.exe**并回车，你就进入了交互界面，这时你需要将平板关机，利用上面**进入download**的相关内容进入download模式
* 这时，你命令行应该是一个交互界面，显示**BROM>** 字样，这便代表着你成功进入download模式
* 如果你是T10/X2P/X3P/C6/T20等机型(芯片为类型1)，你可以从本教程目录下的**fdls.7z** 中获取两个fdl文件用来进入fdl2交互界面，你需要把两个fdl文件放在SPRD目录下，然后在命令行里执行**fdl fdl1.bin 0x5500 （回车） fdl fdl2.bin 0x9efffe00 (回车) exec (回车**以进入fdl2命令行模式；如果是新机型（以C10，S30之类新展讯机型为代表的类型2机型），自求多福，或者使用A10(也为TX20平台)的已Root全量包
* 此时，如果能正常运行上面的命令，便会自动显示分区表，告知你各分区大小，方便你后面的提取
* 输入**read_part 分区名 0 分区大小** 以读取**vendor,system**分区，速度还是比较快的，6-8MB/s
* 提取完进入下一步：修改system，vendor分区以root

### 修改system(本文默认你已配置好WSL2 Ubuntu环境：包括不带任何错误的安装好、启动过至少一次、创建了自己的账户及密码)

**2024.7.27:现在可以选择使用[一键脚本](https://github.com/KDXF-BOOM/studentpad-research/blob/System-Image-Patch-Script/patch_root_adb.sh)进行半自动化修补，但你需要自行安装unzip软件包，但请注意，本脚本默认在root用户下运行且提示语为英文，你如果要使用需要有一定英文基础 or 会用翻译软件**

**如果出现operation not premitted，你需要创建虚拟磁盘然后挂载到Linux下并移动文件到虚拟磁盘下/使用chattr命令更改文件属性**

**如果出现空间不够，请自行删除system分区下app中TyeBroswer/IFlyOTA/IFlyCleanMaster/IFlyPdfReader等文件夹然后再次运行脚本**

你需要在镜像目录的路径栏中输入 `wsl -d Ubuntu -u root`并回车即可进入Linux环境，并异步执行以下命令：

```
mkdir system
sudo mount -o rw system.img system
sudo cp system-root/bin/magisk  system/system/bin/magisk
sudo cp system-root/bin/magiskpolicy system/system/bin/magiskpolicy
sudo cp system-root/init/magisk.rc system/system/etc/init/magisk.rc
sudo cp -r system-root/magisk system/system/etc/init
sudo chmod 0700 -R system/system/etc/init/magisk 
sudo chown -R 0 system/system/etc/init/magisk
sudo chcon -R -h u:object_r:system_file:s0 system/system/etc/init/magisk
cd system/data
mkdir adb
cd ..
cd ..
sudo cp -r system-root/data-magisk system/data/adb/
```

#### 修改bootanim.rc（必做）：

```
sudo nano system/system/etc/init/bootanim.rc
```

用键盘将光标移动在最底下并按几下回车，然后向其中复制以下内容
Tips:把鼠标放到代码框右上角，你会发现有一个复制的按钮，按一下就好，然后在命令行中使用右键进行粘贴即可

```
on post-fs-data
    start logd
    exec u:r:su:s0 root root -- /system/etc/init/magisk/magiskpolicy --live --magisk
    exec u:r:magisk:s0 root root -- /system/etc/init/magisk/magiskpolicy --live --magisk
    exec u:r:update_engine:s0 root root -- /system/etc/init/magisk/magiskpolicy --live --magisk
    exec u:r:su:s0 root root -- /system/etc/init/magisk/magisk64 --auto-selinux --setup-sbin /system/etc/init/magisk /sbin
    exec u:r:su:s0 root root -- /sbin/magisk --auto-selinux --post-fs-data

on nonencrypted
    exec u:r:su:s0 root root -- /sbin/magisk --auto-selinux --service

on property:vold.decrypt=trigger_restart_framework
    exec u:r:su:s0 root root -- /sbin/magisk --auto-selinux --service

on property:sys.boot_completed=1
    mkdir /data/adb/magisk 755
    exec u:r:su:s0 root root -- /sbin/magisk --auto-selinux --boot-complete
   
on property:init.svc.zygote=restarting
    exec u:r:su:s0 root root -- /sbin/magisk --auto-selinux --zygote-restart
   
on property:init.svc.zygote=stopped
    exec u:r:su:s0 root root -- /sbin/magisk --auto-selinux --zygote-restart
```

在修改完后，你需要用Ctrl+X+Y的组合键来保存你的修改,然后进行下一步

```
sudo gzip system/system/etc/init/bootanim.rc bootanim.rc.gz
sudo cp bootanim.rc.gz system/system/etc/init/bootanim.rc.gz
```

自此完成安装基本root环境，下面到"修改vendor"前都是一些优化步骤

#### 1.卸载系统更新(如果你不想进系统后被人误操作导致砖机的话)

```
cd system/system/app/
sudo rm -rf IFlyOTA
```

#### （可选但非常推荐的步骤）：2.开启adb

Tips：如果你前一步"卸载系统更新"也进行了，那你需要先执行三次 `cd ..`

```
nano system/system/build.prop
```

用键盘将光标移动在最底下并按几下回车，然后加入以下内容（Tips:把鼠标放到代码框右上角，你会发现有一个复制的按钮，按一下就好，然后在命令行中使用右键进行粘贴即可）：

```
persist.service.adb.enable=1
persist.sys.usb.config=diag,adb,mtp
ro.sys.usb.default.config=diag,adb,mtp
```

在修改完后，你需要用Ctrl+X+Y的组合键来保存你的修改，然后就完成了改system的步骤

#### 修改vendor分区

```
mkdir vendor #如果镜像文件目录下已经有这个vendor文件夹就不用执行该步
wget https://kdxf.work/d/magisk_vendor.zip
unzip magisk_vendor.zip
chmod +x ./libmagiskinit.so
sudo cp vendor/etc/selinux/precompiled_sepolicy sepol.in
./libmagiskinit.so --patch-sepol sepol.in sepol.out
sudo cp sepol.out vendor/etc/selinux/precompiled_sepolicy
sudo gzip -k vendor/etc/selinux/precompiled_sepolicy
```

#### 取消挂载修改过的分区：

```
sudo umount system.img
sudo umount vendor.img

```

#### 刷入system，vendor（spd_dump窗口下

```
write_part system system.img
write_part vendor vendor.img
reset
```

注：reset命令是重启的意思，如果你还要刷其他分区，你刷完再执行reset

### 开机后步骤

#### 安装爱玩机工具箱

电脑在解bootloader那一步用的文件夹里打开cmd
输入

```
adb devices
adb install 你爱玩机的apk文件
```

工作模式选择Root，给它授权，然后你就可以使用这里面的安装器了（喜

#### 使用爱玩机工具箱备份全分区（救砖可以使用备份的分区

步骤放图片里了，自己看吧
![1](image/README/1.png)
![2](image/README/2.png)
![3](image/README/3.png)

## 将X3-5G等后续高通机型解bl,Root，开adb

### 大纲

#### 解BL+root

* 装[9008驱动](https://kdxf.work/%E9%AB%98%E9%80%9A%E6%9C%BA%E5%9E%8B/x3-5G/%E9%A9%B1%E5%8A%A8)，下一个[HxD Editor](https://mh-nexus.de/downloads/HxDSetup.zip)
* 进入9008模式(关机，按住音量加再插上数据线，然后就进去了)
* 使用[firehose文件](https://kdxf.work/%E9%AB%98%E9%80%9A%E6%9C%BA%E5%9E%8B/x3-5G/firehose)和[工具箱](https://t.me/tgshaw01 "下不了？你都TM会上Github了，翻墙创个tg号不会？")读取boot_a和boot_b,frp分区(记得备份！！！你砖了我不负责)
* 利用HxD Editor对frp的镜像进行修改，将最后一个字节改为1（修改前好习惯，备份）
* 将修改后的frp通过9008刷入,然后通过rec进bootloader，并输入 `fastboot flashing unlock`解锁bl
* 利用工具箱中功能修补两个boot并刷回去就可以了

#### 开adb

等那几个人搞出来twrp适配再说，到时候直接twrp一刷就行了，挺简单的（

或者你仿照展讯机型教程的试试？就是稍微复杂了些

## 衍生教程

### T20Pro

[激活双系统（即dsu）教程](https://github.com/KDXF-BOOM/studentpad-research/blob/main/t20p.md) ——By[@YedLeo1](https://github.com/YedLeo1)

### C10(Pro)

[Yixiyixi666/KDXF-studypad-C10 (github.com)](https://github.com/Yixiyixi666/KDXF-studypad-C10) ——By[@Yixiyixi666](https://github.com/Yixiyixi666)

### 通用教程（不一定是全机型）

[OTA全量包、差分包及如何利用的一系列方法(全机型通用)](https://github.com/KDXF-BOOM/studentpad-research/blob/main/guide_OTA_full_Get.md)——By[@KawaiiSparkle](https://github.com/KawaiiSparkle)

[替换内置PackageInstaller与DefaultContainerService程序(文件仅适用于安卓9，安卓11、12机型需要自行找对应版本的文件进行替换)](https://github.com/KDXF-BOOM/studentpad-research/blob/main/replace_the_original_installer.md)——By[@KawaiiSparkle](https://github.com/KawaiiSparkle)

## 附录：一些资源及其使用方法/作用

#### SPD_Driver

Link：[Download SPD Driver R4.20.4201 (UniSoc Driver) (androiddatahost.com)](https://androiddatahost.com/dsa6h)
国内网盘分流：[到“紫光通用”目录下下载“紫光驱动”即可](http://qutick102.ysepan.com/)

作用：ADB驱动+展讯下载模式的驱动

使用方法：下载安装即可

注：macOS、Linux不需要安装这个东西

#### spd_dump及CVE-2022-38694_unlock_bootloader项目

Link：[GitHub - TomKing062/CVE-2022-38694_unlock_bootloader](https://github.com/TomKing062/CVE-2022-38694_unlock_bootloader)
Spd_Dump工具下载：[到“紫光通用”目录下下载日期最新的spd_dump_dev版本](http://qutick102.ysepan.com/)

#### 学习机配置及目前玩机进度一览表（包括Root，TWRP，GSI）

| 学习机型号 | 系统版本    | 运/储存配置 | soC型号                   | 是否可以安装第三方APP | 是否可以进行Root                                 | 备注                                                                                                                                                       | TWRP                   | GSI支持                 |
| ---------- | ----------- | ----------- | ------------------------- | --------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------- | ----------------------- |
| T10        | Android 9.0 | 8+256       | 展讯T710类型1             | Y                     | Y                                                |                                                                                                                                                            | 已成功编译，但刷不进去 | 暂时还不行              |
| X2Pro      | Android 9.0 | 4+128       | 展讯T710类型0             | Y                     | Y                                                |                                                                                                                                                            | 没设备树               | OK                      |
| X3Pro      | Android 9.0 | 8+256       | 展讯T710类型0             | Y                     | ？                                               | 操作人员刷了C6的系统，后面就没进展了                                                                                                                       |                        | Half-OK，需要修改vendor |
| T20        | Android 9.0 | 8+256       | 展讯T710类型1             | ？                    | ?                                                | 卡在进download，操作人员的机器没有反应                                                                                                                     |                        | 未知                    |
| X1Pro      | Android 9.0 | 4+32        | 高通450/625               | Y                     | Y                                                | Root教程在[X1 PRO - 研究导航 - 小白向supersuroot.github.io](https://supersuroot.github.io/)                                                                   |                        | 未知                    |
| C6         | Android 9.0 | 4+64        | 展讯T710类型1/0           | Y（毕业后官刷）       | N（有多人测试后反应不行）                        | 校园版请毕业后再折腾                                                                                                                                       |                        | OK                      |
| T20Pro     | Android 12L | 8+512       | （瑞芯微）RK3588          | Y                     | ？（可以使用自带root的DSU镜像）                  | 神金，用你RK的工具根本找不到那个frp分区，uboot被删减了，avb是不存在的。我们现在只能希望有个人主动试试刷入magisked_boot来看看能不能root。总之，最抽象的一集 | 已成功编译，但刷不进去 | 未知                    |
| X3-5G      | Android 11  | 6+256       | 高通 骁龙750G             | Y                     | Y（2024.5.19更新，目前已经可以了）               | 最有乐子的一集，9008进入方式与展讯下载一模一样,是通过修改frp分区最后一个字节为1来进行允许解锁oem进而解锁bl                                                 |                        | 未知                    |
| C10（Pro） | Android 9.0 | 4+128       | 展讯T710类型2(暂时不可解) | ？（老版本可以）      | N（无法使用spd_dump,加载值未知，试图开发新方法） | 未知                                                                                                                                                       |                        | 未知                    |

群聊：766720774(QQ),看管理心情踢人

## Contributors | 贡献者名单

[@KawaiiSparkle](https://github.com/KawaiiSparkle)/[@qwqlemon2333](https://github.com/qwqlemon2333)一起编写了伪造apk更新包教程+Root教程
[@KawaiiSparkle](https://github.com/KawaiiSparkle)研究了IFlyOTA，并编写了相关文档
[@Tomking062](https://github.com/Tomking062)提供了Root的思路、spd_dump工具(改进版本)
[@whhh233](https://github.com/whhh233)为我们免费提供了网盘来存放文件
[@WalleoAndrew](https://github.com/WalleoAndrew)最初开始搞科大AI学习机解除安装限制的人
[@YedLeo1](https://github.com/YedLeo1)正在研究T20Pro解锁BL，他编写了T20Pro的DSU食用方法，在[本项目下的t20p.md中](https://github.com/KDXF-BOOM/studentpad-research/blob/main/t20p.md)
[@KawaiiSparkle](https://github.com/KawaiiSparkle)/[@LYao2514](https://github.com/LYao2514)创作了一键自动patch系统分区的脚本

## 常见Q&A

Q1:如何进入榜单？
A1:你得做足够大的贡献才行
Q2:为什么大佬不愿意帮我?
A2:首先，**新手**不代表你可以**不动脑子**,我们更希望的是你能通过[菜鸟教程](https://www.runoob.com/linux/linux-tutorial.html)、[Bilibili](https://www.bilibili.com)、[Bing](https://www.bing.com/)等网址自行解决问题，至于一些英文的东西需要自行翻译
Q3:如何鉴别安卓版本(9 or 11/12)
A3:注意时间的位置(左上角 or 右上角)，在左边即为11/12；去(`设置——存储`点击"系统")，这样可以得到安卓版本号。
Q4:如何鉴别设备soC厂商？
A4:和各机型的"下载模式"有关,这是所有的提示
