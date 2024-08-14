# OTA全量包 获取教程

## 请求更新包

打开爱玩机工具箱APP，点到导航，往下滑找到"系统相关",点进去，下滑找到"SetEdit-安卓设置项修改"并进入，在上方选择“PROP”一栏，然后搜索找到 `ro.build.display.id`。这个值很有意思，它决定了你设置app和系统升级app显示的系统版本号

然后你就可以打开系统升级app，点"版本更新记录"，找到最底部的那个版本号（比如T10第一个被记载的版本号是IFLY_V1.02.6），那就把这个版本号填到刚刚那个 `ro.build.display.id`里面并保存，然后将系统升级的进程从后台杀死再打开，然后你便发现系统升级的"当前版本"的值也变掉了，并推送一个比以往任何更新都大的一个更新。

## root后获取方式

将得到的zip包从/cache/ota中adb pull到电脑上，打开，你会发现这个包三个以new.dat.br格式结尾的文件都大于1G……挺像你在TWRP里刷的那些包？没错，这确实是OTA全量包

## 没有Root的解决方法

* （展讯机型）如果你的机型支持spd_dump（即fdl fdl1.bin时未出错），你可以直接使用最新版的spd_dump提取cache分区（大小都为10G），具体命令请按照主教程底下的格式自行编写，然后拿7zip打开，打开ota目录，里面那个.zip就是
* （高通机型）拿吸盘和热风枪将屏幕拿下来，然后找到主板，短接上面的一些针脚（什么boot正负之类的，参考Lenovo TB-J607Z去）并插入线进入9008，然后QPT，QFIL，启动！（剩下的我认为不用教了）
* （瑞芯微机型）参考T20Pro那篇教程：[studentpad-research/t20p.md at main · KDXF-BOOM/studentpad-research (github.com)](https://github.com/KDXF-BOOM/studentpad-research/blob/main/t20p.md)，搞个DSU进GSI系统，然后adb root，后面的步骤与root后无异，故也不展开了
* 能连的上adb的机型（实在没辙用这个）：在重新打开系统升级前在电脑上执行 `adb logcat > logcat.txt` ，然后打开系统升级app，检查修改的版本无误后点击下载，下了大概10%后就暂停，然后电脑那边 `Ctrl+C` 就直接停止logcat了，打开那个logcat.txt，搜 `user.zip` 然后你就得到了更新包的那一整串链接，然后你就拿Xdown之类的下载器下去吧，记得将UA改为你板子在[UserAgent分析和查询 浏览器UA分析 UA查看 iP138在线工具](https://tool.ip138.com/useragent/)中得到的那个，我认为这样就差不多了

## 碎碎念
笔者编这个教程时，最新版本是1.08.1，然后我往下一个一个试，发现改到1.07.0或更低版本时（这两个版本相差时间大约有一年了），系统更新下载全量包才会触发，我认为这可能说明了科大讯飞更新服务器仅保存1年之内的所有拆分包，超过发布时间一年的会自动被删除