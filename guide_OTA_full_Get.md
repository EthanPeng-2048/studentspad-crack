# 请求、获取OTA更新包及利用其内容的一系列方法
## 注意：本教程默认您使用的是Debian系的Linux发行版(比如Ubuntu\Debian)
## OTA全包(文件名前缀为"full",且利用7zip打开有.br文件的压缩包)
### 请求方法

#### root后的方法
打开爱玩机工具箱APP，点到导航，往下滑找到"系统相关",点进去，下滑找到"SetEdit-安卓设置项修改"并进入，在上方选择“PROP”一栏，然后搜索找到 `ro.build.display.id`。这个值很有意思，它决定了你设置app和系统升级app显示的系统版本号

然后你就可以打开系统升级app，点"版本更新记录"，找到最底部的那个版本号（比如T10第一个被记载的版本号是IFLY_V1.02.6），那就把这个版本号填到刚刚那个 `ro.build.display.id`里面并保存，然后将系统升级的进程从后台杀死再打开，然后你便发现系统升级的"当前版本"的值也变掉了

#### 没有Root的方法

等更易于使用的脚本出来（

### 获取方法

#### root后获取方式

将得到的zip包从/cache/ota中adb pull到电脑上（要装adb root模块

or

使用一些第三方文件管理器将zip包从/cache/ota复制到本地存储，然后你自由发挥（

#### 没有Root的解决方法

等脚本，或者你有水平就自己构建一个.需要拿到两种不同机型(如T10\X2Pro 与 X3Pro\T20\T20Pro，建议是T10\X2pro与X3Pro)的最新版本的IFLYOTA.apk 原因是这个用了两种不一样的OTA包Request方式

### 利用方法

#### 从完整OTA包得到数个系统分区镜像
* 去安装[DNA3工具箱](https://github.com/ColdWindScholar/D.N.A3)，把你OTA包(.zip格式文件)直接复制到DNA3的run.py所在目录下。运行DNA3，选解压，就能看到你的全量包了，选择你全量包对应的选项，它便会自行开始进行镜像的转换，你只需选择开启静默它便会自行合成镜像。
至于得到的镜像，如果你是安卓9机型，你就不需要考虑super分区之类的；反之，如果你是安卓11，12机型，我强烈建议你先自行合成一个super分区镜像，这样是为了砖以后刷进去救砖。合成的super的具体大小需要你自己通过高通9008模式或者瑞芯微loader模式读取分区表得。总之你可以**通过修改vendor,system**来进行root、当然，如果你是安卓11，12机型，你可以直接修补boot镜像并刷回去(适用于这种的情况要么是你解了bl，要么就是你boot分区根本不检验，整套分区的avb是失效的某个机型)

## 非全量包(aka. inc包/差分包)
### 识别
看文件名是否遵循如下标准
```
inc-<更新后版本号>-from-<更新前版本号>.zip
```
### 请求方法
修改请求的当前版本号为你全量包版本号
### 利用方法
全量包镜像合并差分包[.dat .p文件]=>得到升级后的镜像
* 这里用了一个项目[imgpatchtools](https://github.com/erfanoabdi/imgpatchtools)，它可以在linux下模拟OTA升级过程，只不过需要你自行编译(git链接自行加速，这里我就不处理了)
Debian下编译的命令如下：
```
  apt install make git libbz2-dev libssl-dev zlib -y
  git clone https://github.com/erfanoabdi/imgpatchtools.git ; cd imgpatchtools ; make ; cp -r bin/ ../ ; cd .. ; cp bin/* ./ ; rm -rf imgpatchtools ; rm -rf bin ; chmod +x ./* 
```
修补OTA镜像命令
```
BlockImageUpdate system.img system.transfer.list system.new.dat system.patch.dat
BlockImageUpdate product.img product.transfer.list product.new.dat product.patch.dat
BlockImageUpdate vendor.img vendor.transfer.list vendor.new.dat vendor.patch.dat
```

```
其他分区：
BlockImageUpdate 名称.img 名称.transfer.list 名称.new.dat 名称.patch.dat
Boot分区：
Applypatch boot.img - <After_patching_sha1> <After_patching_size> <Before_patching_sha1> boot.img.p
```
注：如果你要修补OTA前boot镜像，建议先使用`sha1sum boot.img`来计算原镜像sha1(算出来的这个值是`Before_patching_sha1`变量的值)，然后你在update-script中查找`boot.img.p`，能找到两个sha1值,你需要**复制那个异于你算出来的值的值**，作为`After_patching_sha1`变量的值，至于那个`After_patching_size`的值，你需要从`After_patching_sha1`变量前面找到
(这里我参考的是T20Pro差分包的情况，普通安卓12也是这个情况)
后面你就把变量值填入，不要有`<>`符号

## 碎碎念
 为啥会发现这个呢？这就要说到另一个东西了。
 很久之前，我们那边一个大佬发现了系统安装器有黑名单这种东西，并且发现只需临时删除黑名单，并且把系统安装器和软件权限帮助程序替换，就能自由安装某四字国民5v5MOBA游戏，给我留下了深刻的印象

 后面我在我第一次尝试获得升级包时，我刷入了旧版1.06.9的系统，因为印象中我板子曾在被售后过恢复到一个较早的系统版本后收到更新。不出我所料，又被推送了一次，大小我记为`size1`。后面又刷了一个更早版本的系统，是1.04.9，我记更新包大小为`size2`，以此类推……后面我就嫌这样太麻烦了，关键得有更多更老的版本备份，我手上最老的就1.04.9，再下没了。
 我得想办法获取更老的版本。后面我于是想到了之前安装器那个方法，便试着去修改系统版本，结果就得到了这个结论，属于偶然发现（）