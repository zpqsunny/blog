---
title: Synology VideoStation ffmpeg wrapper with DTS, EAC3 and TrueHD support
date: 2022-02-16 21:15:16 +0800
categories: [Linux]
tags: [Synology, VideoStation, ffmpeg]
---


> 原文: [https://zhuanlan.zhihu.com/p/393311059](https://zhuanlan.zhihu.com/p/393311059)


1. 安装ffmpeg
套件中心->设置->套件来源->新增
http://packages.synocommunity.com/
国内源: https://spk7.imnks.com/
搜索安装`ffpmeg`

2. 通过SSH连接群晖,并输入sudo -i切换至root用户模式(Tips:SSH需要进入控制面板->终端机和SNMP->终端机->启用SSH功能)

3. 依次执行以下内容脚本
``` shell
#备份 VideoStation's ffmpeg
mv -n /var/packages/VideoStation/target/bin/ffmpeg /var/packages/VideoStation/target/bin/ffmpeg.orig
#下载ffmpeg脚本
wget -O - https://gist.githubusercontent.com/BenjaminPoncet/bbef9edc1d0800528813e75c1669e57e/raw/ffmpeg-wrapper > /var/packages/VideoStation/target/bin/ffmpeg
#设置脚本相应权限
chown root:VideoStation /var/packages/VideoStation/target/bin/ffmpeg
chmod 750 /var/packages/VideoStation/target/bin/ffmpeg
chmod u+s /var/packages/VideoStation/target/bin/ffmpeg
# 备份VideoStation's libsynovte.so
cp -n /var/packages/VideoStation/target/lib/libsynovte.so /var/packages/VideoStation/target/lib/libsynovte.so.orig
chown VideoStation:VideoStation /var/packages/VideoStation/target/lib/libsynovte.so.orig
# 为libsynovte.so 添加 DTS, EAC3 and TrueHD支持
sed -i -e 's/eac3/3cae/' -e 's/dts/std/' -e 's/truehd/dheurt/' /var/packages/VideoStation/target/lib/libsynovte.so
#备份CodecPack的ffmpeg41 这两端命令行重要 很多文章/脚本都缺少这两个命令行
cp /var/packages/CodecPack/target/bin/ffmpeg41 /var/packages/CodecPack/target/bin/ffmpeg41.bak
#链接ffmpeg解码模块
cp /var/packages/VideoStation/target/bin/ffmpeg /var/packages/CodecPack/target/bin/ffmpeg41
```
4. 重新启动Video Station，让ffmpeg与Video Station的关联生效

6. 如何还原和卸载?
``` shell
#恢复之前备份的 VideoStation's ffmpeg, libsynovte.so, ffmpeg41文件
mv -f /var/packages/VideoStation/target/bin/ffmpeg.orig /var/packages/VideoStation/target/bin/ffmpeg
mv -f /var/packages/VideoStation/target/lib/libsynovte.so.orig /var/packages/VideoStation/target/lib/libsynovte.so
mv -f /var/packages/CodecPack/target/bin/ffmpeg41.bak /var/packages/CodecPack/target/bin/ffmpeg41
```
