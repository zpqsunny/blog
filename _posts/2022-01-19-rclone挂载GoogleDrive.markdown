---
title: rclone挂载GoogleDrive
date: 2022-01-19 10:55:00 +0800
categories: [Linux]
tags: [GoogleDrive, rclone]
---

> 原文: [https://www.unvone.com/69270.html](https://www.unvone.com/69270.html)


## 安装

环境Centos7
```shell
curl https://rclone.org/install.sh | sudo bash
yum install fuse -y
```

## 开始配置
``` shell
rclone config
```
输入 **n** 新建
``` shell
No remotes found - make a new one
n) New remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
n/r/c/s/q> n
```
然后回车输入一个name(名字)，例如：这里我创建的名字为 【GoogleDrive】
``` shell
name> GoogleDrive
```
输入**XX**序号
``` shell
Type of storage to configure.
Choose a number from below, or type in your own value
[snip]
XX / Google Drive
   \ "drive"
[snip]
Storage> XX
```
接下来会叫选填 Google Application Client ID 和，这里 2 项都留空回车即可：
``` shell
Option client_id.
Google Application Client Id
Setting your own is recommended.
See https://rclone.org/drive/#making-your-own-client-id for how to create your own.
If you leave this blank, it will use an internal key which is low performance.
Enter a string value. Press Enter for the default ("").
client_id> 直接回车
```
``` shell
Option client_secret.
OAuth Client Secret.
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret> 直接回车
```
rclone 在请求从驱动器访问时应使用的范围，这里选 1 即可
``` shell
Option scope.
Scope that rclone should use when requesting access from drive.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value.
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1
```
选填文件夹 ID 和帐户凭据 JSON 文件路径，这里我都留空回车的
``` shell
Option root_folder_id.
ID of the root folder.
Leave blank normally.
Fill in to access "Computers" folders (see docs), or for rclone to use
a non root folder as its starting point.
Enter a string value. Press Enter for the default ("").
root_folder_id> 直接回车
```

``` shell
Option service_account_file.
Service Account Credentials JSON file path.
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Leading `~` will be expanded in the file name as will environment variables such as `${RCLONE_CONFIG_DIR}`.
Enter a string value. Press Enter for the default ("").
service_account_file> 直接回车

```
问你是否要编辑高级配置，这里我选择 N

``` shell
Edit advanced config?
y) Yes
n) No (default)
y/n> n
```
是否自动远程配置，这里选择 N，因为你在手动远程配置

``` shell
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine

y) Yes (default)
n) No
y/n> n
```
接下来会给出Google Drive的授权地址，把它复制到浏览器打开，按提示登陆，复制获取到的代码，然后返回 SSH 粘贴后回车

``` shell
Option config_verification_code.
Verification code
Go to this URL, authenticate then paste the code here.
https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=202264815644.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&state=WXR8NtP7PdDYXfw-ftOFNw
Enter a string value. Press Enter for the default ("").
config_verification_code>
```

问你是不是将其配置为团队网盘，这里选 n
``` shell
Configure this as a team drive?
y) Yes (default)
n) No
y/n> n
```

列出你上面修改的配置信息，问你是否删除、编辑或者确定配置信息，这里选择确定 y

``` shell
y) Yes this is OK (default)?
e) Edit this remove
d) Delete this remove
y/e/d> y
```

## 挂载目录

演示挂载在可道云盘的目录，我要挂载到 /www/wwwroot/pan.unvone.com/data/User/admin/home/Drive

挂载命令1（挂载整个谷歌云盘）：

``` shell
rclone mount GoogleDrive: /www/wwwroot/pan.unvone.com/data/User/admin/home/Drive --allow-other --allow-non-empty --vfs-cache-mode writes
```

挂载命令2（只挂载谷歌云盘的某个目录-backup目录）：
``` shell
rclone mount GoogleDrive:backup /www/wwwroot/pan.unvone.com/data/User/admin/home/Drive --allow-other --allow-non-empty --vfs-cache-mode writes
```

GoogleDrive 为 Rclone 的前面配置名称，比如你在创建配置 rclone 的时候 Name 填的GoogleDrive，

/www/wwwroot/pan.unvone.com/data/User/admin/home/Drive为VPS本地路径；

挂载只要几秒钟，但终端不会返回成功信息，关闭 SSH 重连即可。

## 设置开机自动挂载
新建一个/usr/lib/systemd/system/rclone.service 文件内容为
``` shell
vim /usr/lib/systemd/system/rclone.service
```
```
[Unit]
Description=rclone

[Service]
User=root
ExecStart=/usr/bin/rclone mount GoogleDrive: /www/wwwroot/pan.unvone.com/data/User/admin/home/Drive --allow-other --allow-non-empty --vfs-cache-mode writes
Restart=on-abort

[Install]
WantedBy=multi-user.target
```
重载daemon，让新的服务文件生效
``` shell
systemctl daemon-reload
```
启动 rclone
``` shell
systemctl start rclone
```
设置开机启动
``` shell
systemctl enable rclone
```
