
本文主要介绍抓取报文的工具。  
<!--more-->

# tcpdump
```shell
# 保存为cap
tcpdump -Xnnlps0 -w pkts.cap -i any port 80 and host 125.78.252.151

# 明文查看
tcpdump -Xnlps0 -i any dst port 28000 > pkts.txt
```
`-i` 指定网卡，`-w` 指定输出文件，`-X` 打印报文内容。  

# dtk
http://code.google.com/p/dtk/downloads  
支持报文解析、重放。  

# wireshark
https://www.wireshark.org/download.html

# Fiddler
https://www.telerik.com/download/fiddler
