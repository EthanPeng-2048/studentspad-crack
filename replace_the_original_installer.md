# 替换安装器

在[本项目项目System-Patch-Script分支](https://github.com/KDXF-BOOM/studentpad-research/tree/System-Image-Patch-Script)有一个文件夹，叫 `pm_change`，里面有用来替换用的安装器与权限管理程序，你需要在网站上点绿色按钮，然后点 `Download Zip`，会下载一个压缩包，你需要将其中 `DefaultContainerService`与 `PackageInstaller`两个文件夹 解压到与下文 '你自己创建的文件夹' 下。

## 注意事项

* 请先获得你家长的同意再进行操作，否则因为未得到许可导致出现的一切人员损失、机器损坏本教程不负责😋
* 我默认你为Win10/11系统，且已经完成**在'程序与功能'中启用了'虚拟机平台'与'适用于Linux的Windows子系统'、自行完成在微软商店(Microsoft Store)中安装Ubuntu并启动，创建账户与密码**的操作，否则请**先去完成以上步骤**再看下面部分
* 我默认你已经提取完了system分区镜像并**备份过了**，否则因你自己的不当操作而造成的一切损失由你自行负责

## 正式操作

### 替换

* 将提取出的system镜像移动到你**自己创建的文件夹(这里为方便表述命名为f1)**，在 `f1`下创建一个新文件夹，命名为'root' .在一开始创建的文件夹窗口的空白部分使用'Shift+右键'打开菜单，选择'在此处打开Linuxshell(L）'打开窗口
* 使用 `sudo mount -o rw 你的system镜像文件名称 root/ `命令进行挂载system分区
* 使用 `sudo rm -rf root/system/priv-app/PackageInstaller `与 `sudo rm -rf root/system/priv-app/DefaultContainerService`删除原来系统中内置的安装器
* 使用 `sudo cp -r PackageInstaller/ root/system/priv-app/PackageInstaller/`与 `sudo cp -r DefaultContainerService/ root/system/priv-app/DefaultContainerService/` 进行安装器替换。

### 保存更改

* 使用 `sudo umount 你的system镜像文件名称`来卸载分区
* 让板子进入下载模式，进行刷机前的准备工作：打开spd_dump_interactive.exe、加载fdl1.bin到0x5500、加载fdl2.bin到0x9efffe00、输入 `exec` 并回车。此时交互式刷机工具应为 `fdl2 > `则完成了该步骤
* 执行 `w system 你的system镜像文件名称` 命令来刷入镜像
* 教程结束
