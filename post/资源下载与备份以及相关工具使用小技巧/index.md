
本文主要记录如何更快地下载我们需要的资源并做好备份。  
<!--more-->

# 下载
## 资源网站
链接可能失效(先确认科学上网)，搜关键字即可。  
- [人人影视](https://yyets.dmesg.app)  
- [rarbg](https://www.rarbgtor.org)  
- [eztv](https://eztv.re)  
- [nyaa](https://nyaa.si)  
- [动漫花园](https://share.dmhy.org)  
- [字幕库](http://zmk.pw)  
- [伪射手](https://assrt.net)  
- [琉璃神社](https://www.liuli.cat)  
- [新次元](https://newacg.moe)  

视频资源可以下载后挂字幕观看。  

## 正规资源
正规的资源可以通过网盘的转存(秒传)、离线下载以及下载工具(如迅雷)快速地下载。  
当然要想速度更快就需要开通会员了。  

## 非正规资源
非正规资源下载大概率不能使用上面的方法，很容易遭遇封锁不让下载。  
目前推荐使用 `aria2` 或者 `youtube-dl` 两种下载工具，有其他好用的工具大家也可以在下面留言。  
工具的安装和使用这里就不赘述了，网上很多。这里分享些小技巧：  
- VPS硬盘空间不够怎么办  
  以磁力链为例，aria可以选择下载哪些文件。AriaNg上面先暂停任务就可以在文件列表里选择了。  
- 浏览器可以下载，aria下载不了？  
  常见于HTTP下载，一般是缺少必要的HTTP首部信息(比如说cookie)。  
  这个需要一点前端调试技巧来获取这些信息，篇幅有限就不具体展开了。  
  获取必要信息后就可以在aria中HTTP设置的 **自定义请求头** 填好，然后创建下载任务。  
  举例来说，UC网盘的资源就可以使用这个技巧快速下载。  

另外建议在VPS上面安装、使用这些工具，带宽更大而且方便后续的备份操作。  

### 国外网盘
这里主要指的是那些不充会员就有各种限制的国外网盘。  
可以通过网盘中转站(如`VVLoad`, `木薯牛`, `帮你下`等)代为下载，之后获取不限速的链接进行下载。  
虽然中转站是收费的(1GB1元左右)，不过免除了注册、充值多个网盘会员的开销。有需要的可以尝试。  

# 备份
备份可以同步到硬盘(或NAS)和网盘。前者比较简单不多说。  
百度云盘不是很推荐，一方面是有被和谐的风险，一方面是限速。一定要用的话就存带密码的压缩文件。  
推荐使用 `OneDrive` 和 `Google Drive` ，不限速、不随便封锁资源(特别是不分享的情况下)。  
容量方面，OneDrive有家庭版、教育版E3以及开发者E5等。Google Drive也有教育版、无限容量版。  
褥羊毛的教程很多，这里也不赘述。  
备份资源到网盘，可以使用 `rclone` 工具，命令使用 `rclone move` 或者 `rclone copy` 都可以。  
```shell
rclone move src:path dst:path -P
# 后台同步
nohup rclone sync gdrive:/root onedrive:/root &
```
另外与aria2配合使用就可以实现下载完自动同步到网盘的效果。网上的一键安装脚本就带有这个功能。  
rclone还可以挂载网盘到本地文件系统，配合使用 `Plex`、`Emby`、`VLC`任意软件就可以播放网盘的视频。  
轻松实现个人媒体中心，再也不用担心硬盘空间不够了~ :heart_eyes:  
```shell
rclone cmount \
	--read-only \
	--disable About \
	--buffer-size 32M \
	--dir-cache-time 6h \
	--vfs-read-chunk-size 8M \
	--vfs-read-chunk-size-limit 128M \
	onedrive: ~/onedrv

# --disable About 选项是Mac系统用的，其他系统可以不加

rclone mount \
	--daemon \
	--buffer-size 32M \
	--vfs-cache-mode full \
	--vfs-cache-max-size 10G \
	--vfs-read-chunk-size 8M \
	--vfs-read-chunk-size-limit 128M \
	gdrive: ~/gdrv

fusermount -u ~/gdrv
```
上面的数值仅供参考，可以根据自己的情况适当调整。  

## 国内网盘
像百度云盘、阿里云盘、天翼云、和彩云等等做个备份网盘还是可以的。  
国内网盘不建议作为主力网盘，审查、限速、不定期回收容量等等都是硬伤。  

## OneDrive
开发者E5挺好申请，结合`AutoApiSecret`保持E5开发活跃、提高续期概率(每3个月)。  
OneDrive国内也可以下载，节省科学上网流量。  

## 谷歌Team Drive
无限容量、不限速的免费网盘，结合[AutoRclone](https://github.com/xyou365/AutoRclone)多重备份、再叠加新秀[IPFS](https://ipfs.io)分布式存储，真香！  
网上很多免费申请的地址都挂了，这里发个小福利——[蓬莱岛](https://penglai.ga)，目前已经撸了8个Team Drive:sunglasses:。  
AutoRclone利用多个谷歌服务帐号(大数据量)同步多个Team Drive，一般同步2个翻车的概率就比较低了。  
另外谷歌的[colab](https://colab.research.google.com)可以白嫖(相当于免费的VPS)，用来AutoRclone效果更佳~  
IPFS不是永久存储(收费的filecoin白嫖党不考虑)，不过把ipfs数据存到Team Drive可变相实现永久存储:sunglasses:。  

