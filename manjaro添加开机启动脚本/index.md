# Manjaro添加开机启动脚本


如何给manjaro添加开机启动脚本。

<!--more-->

![](https://wiki.manjaro.org/images/thumb/e/ea/New_logo_tex.png/800px-New_logo_tex.png)


前段时间折腾小黑T450安装了Manjaro系统，又安装了sublime text 3，顺便参考[Installation Package Control指导](https://packagecontrol.io/installation)安装了package control插件，但是很不幸国内的package control页面已经被屏蔽了。因此，需要给package control设置代理，根据[《SublimeText 安装 PackageControl 及 HTTP 代理配置 - Tony的技术笔记 - SegmentFault 思否》](https://segmentfault.com/a/1190000007621085)我看到一个方法是配置完设置之后，在shell中运行的命令

```shell
polipo socksParentProxy=localhost:1080
```

设置一下sublime text 3中的package control的代理就可以使用它安装插件了。但是每次开机之后都要敲入上面的命令，非常麻烦。

## 安装过程

参考官方资料，按照如下的步骤设置了自动启动脚本

1. 新增一个autostart的桌面启动项，这一项开机后会随桌面启动

   ```shell
   cd ~/.config/autostart
   touch AutoExec.desktop
   ```

2. 在创建的`AutoExec.desktop`中写入如下的内容

   ```shell
   [Desktop Entry]
   Type=Application
   Exec="/etc/AutoExec.sh"
   Terminal=yes
   Name=AutoExec
   X-GNOME-Autostart-enabled=true
   ```

3. 创建开机自启动脚本`/etc/AutoExec.sh`，

   ```shell
   sudo touch /etc/AutoExec.sh
   chmod +x /etc/AutoExec.sh
   ```

   写入如下的内容

   ```shell
   polipo socksParentProxy=localhost:1080 &
   ```

   即在开机之后的后台自动启动。

## 参考资料

- [systemd - ArchWiki](https://wiki.archlinux.org/index.php/Systemd)，官方资料
- [[Solved] Execute script on startup / Newbie Corner / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=86815)，论坛资料
- [SublimeText 安装 PackageControl 及 HTTP 代理配置 - Tony的技术笔记 - SegmentFault 思否](https://segmentfault.com/a/1190000007621085)
