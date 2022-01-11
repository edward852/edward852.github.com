
本文主要记录自用VPS的购买与日常使用。  
<!--more-->

# 选购
除了谷歌、亚马逊可以免费试用的服务器外，我购买过 `Vultr`、`Hostwinds` 以及 `BandwagonHOST` 。  
目前使用的是 `BandwagonHOST` ，也就是大家都在用的搬瓦工。其他不是不稳定，就是速度不行。  
机房是走CN2 GT线路的DC3，晚上偶尔还是不太稳定，不过速度还是不错的。  
听说CN2 GIA线路的机房更好，不过有点贵，目前还是先将就着用吧。  
后面可能试下大盘鸡，配合网盘使用看看效果如何。  

# 系统
选择安装较新的系统，确认支持BBR(网络稳定时速度更快)。  
我一般选择Centos 8或者ubuntu18.04。  

# 登录
购买之后，首先配置SSH免密登录，具体看 [这里](/post/ssh免密登录) 。  
配完直接用别名登录就可以了：  
```shell
ssh vps
``` 

# 中文
安装中文语言包，不同系统命令不同。  
```shell
# centos 8
yum install -y langpacks-zh_CN

# ubuntu 18
apt install -y language-pack-zh-hans
```

# BBR
为了最大程度利用带宽，提高传输速度，需要确认开启BBR：  
```shell
sysctl net.ipv4.tcp_congestion_control
# net.ipv4.tcp_congestion_control = bbr
```
没有的话可以，通过以下命令开启：  
```shell
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

# 搭梯子
用的是v2ray，ss稍旧容易被封，trojan较新后面再考虑使用。  
使用一键安装脚本就可以了：  
```shell
# 安装
bash <(curl -s -L https://git.io/v2ray.sh)

# 查看是否正常运行
v2ray status

# 查看订阅地址
v2ray url
```

# 安装aria2和rclone
这两个软件主要用来离线下载和同步网盘。  

## aria2
确认安装`jq`和`wget`之后，使用一键安装脚本就可以了：  
```shell
wget -N git.io/aria2.sh && chmod +x aria2.sh && ./aria2.sh
```
有问题可以通过菜单9查看日志定位错误。  
```shell
./aria2.sh
# 输入9
```
安装之后会显示RPC端口和密钥以及AriaNg链接。  
AriaNg是aria2的web操作界面，可以更方便地管理下载任务和配置。  

## rclone

```shell
curl https://rclone.org/install.sh | sudo bash
```
安装比较简单，主要是配置网盘麻烦一点。  
VPS一般不装图形界面，所以需要在另外一台电脑上通过 `rclone authorize` 命令获取授权token。  

## 下载完自动上传网盘
`/root/.aria2c/script.conf`配置文件更新网盘信息。  
`/root/.aria2c/aria2.conf`配置文件找到`on-download-complete`，`clean.sh`改为`upload.sh` 。  
```shell
on-download-complete=/root/.aria2c/upload.sh
```
重启aria2服务：  
```shell
systemctl restart aria2
```
# python
```shell
# ubuntu 18
apt install -y python3
update-alternatives --install /usr/bin/python python /usr/bin/python3 1
```

# ffmpeg
ffmpeg用来处理视频挺方便的，比如说合并视频和音频。  
```shell
# ubuntu 18
apt install -y ffmpeg

# 合并视频和音频
ffmpeg -i talk.mp4 -i talk.m4a -c copy talk-merged.mp4
```

# youtube-dl
[youtube-dl](https://github.com/ytdl-org/youtube-dl)可以下载许多[视频网站](https://ytdl-org.github.io/youtube-dl/supportedsites.html)的资源，包括油管、优酷、P站等。  
先安装 `python` 和 `ffmpeg` 两个前置依赖项。  
然后按照 [官方说明](https://github.com/ytdl-org/youtube-dl#installation) 操作即可：  
```shell
sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
sudo chmod a+rx /usr/local/bin/youtube-dl
```
下载选项很多，这里以油管为例简单示范一下：
```shell
# 查看视频所有格式
youtube-dl -F video_url

# 下载视频
youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]' -o '01.%(ext)s' video_url 

# 下载视频列表所有视频
youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]' -o 'playlist/%(playlist_index)s.%(ext)s' playlist_url

# 下载视频列表中指定视频
--playlist-items 1,27-30
```
使用的时候注意VPS的硬盘大小和每月流量，适度下载视频资源。  
另外说一句，油管上面有不少国产电视剧的高清资源可以免费下载，遇见合适的赶紧下载同步到网盘吧。  

