
本文主要介绍如何编译安装clang+llvm。  
<!--more-->

# 预编译二进制文件
可以先到 https://github.com/llvm/llvm-project/releases 查看是否有适合你系统的。  
`clang+llvm` 开头的压缩包就是预编译的二进制文件，如果有合适的直接下载安装即可，下文可以忽略。  

如果不要求最新版本，那么也可以使用第三方软件仓库安装：  
```sh
# CentOS7
yum install -y llvm7.0
```

# 下载源码
虽然可以通过Git下载到最新代码，但是不一定能编译通过，又或者可能有bug。  
建议还是到 [releases](https://github.com/llvm/llvm-project/releases) 这里下载 `Source code(tar.gz)` 文件。  

# 安装依赖
- gcc  
  需要新版本的gcc才能编译，可以参考 [编译安装gcc 7.2.0](/post/编译安装gcc7.2.0) 的说明
- cmake  
  到 [cmake官网](https://cmake.org/download) 下载安装即可

# 编译安装
```sh
cd llvm源码解压目录
mkdir Release
cd Release

# 如果需要clangd，则LLVM_ENABLE_PROJECTS加上clang-tools-extra
cmake ../llvm -DCMAKE_C_COMPILER=/usr/local/gcc-7.2.0/bin/gcc -DCMAKE_CXX_COMPILER=/usr/local/gcc-7.2.0/bin/g++ -DCMAKE_CXX_LINK_FLAGS="-Wl,-rpath,/usr/local/gcc-7.2.0/lib64 -L/usr/local/gcc-7.2.0/lib64" -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi;"
make -j4
sudo make install
```
注意根据新版本gcc的安装目录调整上面的 `cmake` 命令参数。  
另外编译比较耗内存，可能会出现内存不足的情况，不带 `-j4` 再次 `make` 即可。  

# 参考链接
- https://llvm.org/docs/GettingStarted.html#requirements
