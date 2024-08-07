# 在Ubuntu20.04中挂载alist服务到本地
## 一、安装alist
在安装alist这里，我使用的是alist官网提供的[一键脚本](https://alist.nn.ci/zh/guide/install/script.html)，这里我就直接照搬了
### 1. 安装
```sh
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s install
```
### 2. 更新
```sh
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s update
```
### 3. 卸载
```sh
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s uninstall
```
### 注意
在安装这里需要使用root用户，或者在bash前面加上sudo
## 二、配置alist
此方面相关教程已有很多，不再赘述
## 三、挂载alist到本地
在这里我选择的是rclone进行挂载
### 1. rclone安装
```sh
wget https://www.moerats.com/usr/shell/rclone_debian.sh && sudo bash rclone_debian.sh
```
### 2. rclone配置
```sh
rclone config
```
按照引导依次选择
`New remote(n)->remote name(alistWebdav)->remote type(52，代表webdav，新版本可能有所不同，注意区分)->webdav address(http://127.0.0.1:5244/dav)->container type(7, others)->username and password`
没什么难度。配置结束后测试是否配置成功：`rclone lsd alistWebdav:`
### 3. 挂载到本地
命令行输入(注意确保本地`/home/user/remote`目录存在，user为自己用户名)
```sh
rclone mount alistWebdav:/ /home/user/remote
```
**注**：若此处报错**failed to mount FUSE fs: fusermount: exec: "fusermount3": executable file not found in $PATH错误**
则使用软链接：
```sh
ln -s /bin/fusermount /bin/fusermount3
```
挂载完毕
## 四、开机自动挂载
这里我是使用注册系统服务的方式完成的这个任务。基本步骤与rclone配置部分基本相同，但需要在root用户下再执行一遍
```sh
rclone config
```
或者也可以在当前用户执行
```sh
sudo rclone config
```
然后编辑`rclone.service`
```sh
sudo vim /etc/systemd/system/rclone.service
```
填入以下内容：
```sh
[Unit]
Description=Rclone
After=network-online.target alist.service

[Service]
Type=simple
ExecStart=rclone mount alistWebdav:/ /home/yundan/remote  --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000
Restart=on-abort
User=root

[Install]
WantedBy=default.target
```
保存退出后使用`sudo systemctl daemon-reload`重新加载配置，之后便可以通过systemctl管理rclone了
**启动**：
```sh
sudo systemctl start rclone
```
**停止**：
```sh
sudo systemctl stop rclone
```
**重启**：
```sh
sudo systemctl restart rclone
```