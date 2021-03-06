
本文主要记录开发机环境搭建的流水。  
<!--more-->

# 创建用户
创建普通用户`edward`, 避免以`root`身份误操作。  
```sh
useradd edward -d /data/home/edward -m
passwd edward
```
用户的home目录需要确保空间足够。  

# 时区
调整时区为东8区。  
```sh
# CentOS, ubuntu
ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

# 国内镜像源
更换软件镜像源为国内镜像源。  
如果不能访问外网的话就使用公司内的镜像源。  

以 [阿里镜像源](https://developer.aliyun.com/mirror/) 为例，CentOS 7如下操作:  
```sh
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

sed -i 's#mirrors.*.com#mirrors.aliyun.com#g' /etc/yum.repos.d/CentOS-Base.repo
sed -i 's#mirrors.*.com#mirrors.aliyun.com#g' /etc/yum.repos.d/epel.repo

yum makecache
```
Ubuntu 18.04如下操作：  
```sh
cat > /etc/apt/sources.list << EOF
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
EOF
```

# 中文支持
安装系统语言支持包、输入法(fcitx, ibus皆可)。  
```sh
# CentOS
yum groupinstall -y "Chinese support"
yum install -y fcitx-pinyin

# ubuntu
sudo apt-get install -y language-pack-zh-hans
# 配置默认语言，选择 zh_CN.utf8
sudo dpkg-reconfigure locales
# 桌面消失处理
rm -fr ~/.cache/sessions/

sudo apt-get install -y fcitx-googlepinyin

# fcitx没有自启动的话
cp /usr/share/applications/fcitx.desktop ~/.config/autostart/fcitx.desktop
```
设置`XMODIFIERS`,`GTK_IM_MODULE`,`QT_IM_MODULE`环境变量。  
```sh
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```
一般放到`/etc/profile`就生效，不行的话试一下`~/.pam_environment`,`~/.xinitrc`,`~/.vnc/xstartup`。  
还有问题可以通过`fcitx-diagnose`定位有问题的地方。  

# dot files
- ~/.bash_profile  
```sh
[[ -r ~/.bashrc ]] && . ~/.bashrc

alias ll='ls -hl --color'

export LC_ALL="zh_CN.utf8" 
export LANG="$LC_ALL"
```

# 字体
安装适合程序员的字体，比如说 `Source Code Pro` 。  
偷懒一点可以通过yum安装:  
```sh
yum install -y adobe-source-code-pro-fonts
```
如果想安装最新的 `Source Code Pro` 或者其他字体：  
```sh
字体复制到 /usr/share/fonts/ 目录
fc-cache -v
```
推荐字体:  
- [Source Code Pro](https://github.com/adobe-fonts/source-code-pro/releases)
- [思源黑体](https://github.com/adobe-fonts/source-han-sans/releases)
- [Fira Code](https://github.com/tonsky/FiraCode/releases)
- [更纱字体](https://github.com/be5invis/Sarasa-Gothic/releases)
- [JetBrains Mono](https://www.jetbrains.com/lp/mono)
- ...

# 代理
有些公司需要通过设置代理才能访问外网。  
可以在 `/etc/profile` 设置代理：  
```sh
http_proxy=dev-proxy.oa.com:8080
https_proxy=dev-proxy.oa.com:8080
no_proxy="localhost,*.oa.com"

export http_proxy https_proxy no_proxy
```

# 基本开发工具
```sh
# CentOS
yum groupinstall -y "Development Tools"

# ubuntu
sudo apt-get install -y build-essential
```
git，cmake, golang, llvm, ccls, emacs, vscode, perf等按需(编译)安装。  

# vnc
具体可以参考 [通过vnc控制linux开发机](/post/通过vnc控制linux开发机/) 的说明。  
通过 [noVNC](https://github.com/novnc/noVNC) 还可以在家通过浏览器远程控制开发机。  

# samba
samba主要作用是方便文件的同步。  
具体可以参考 [CentOS配置samba](/post/centos配置samba/) 的说明。  

# SSH免密登录
具体可以参考 [SSH免密登录](/post/ssh免密登录/) 的说明。  
不过使用终端软件(xshell, PuTTY等)也是可以的。  

# IDE
主要使用vscode、spacemacs、vim，偶尔用用JetBrains系列。  
先记录，后续有空再补充。  

- vscode  
  主力写代码。  
  通过 [vscode remote](https://code.visualstudio.com/docs/remote/remote-overview) 可以在家远程开发。  
- spacemacs  
  目前主要用来写博文，具体可以参考 [通用代码编辑器Spacemacs](/post/通用代码编辑器spacemacs/) 。  
- vim  
  主要用来看服务器的日志，按键还是比较简短、方便的。  
- JetBrains系列  
  小项目还可以，比较耗内存、响应稍慢、要收费，不适合大项目。  

# docker
具体参考 [Docker使用](/post/docker使用笔记/) 的说明。  

