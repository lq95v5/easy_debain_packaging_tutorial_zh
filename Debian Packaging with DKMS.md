DKMS Debian packaging

# 基本说明

Debian DKMS打包是将内核模块的源代码纳入目标系统中DKMS tree，然后在安装时通过apt触发dkms add ;build ;install 等命令自动编译安装内核模块


# DKMS packaging 需要定义的几个文件

***PACKAGE*.install:**

安装时需要内核模块的源码安装到目标系统的/usr/src 目录中的文件夹，文件夹标准格式

`dkms模块名称-上游版本号`


***PACKAGE*.dkms:**


dkms 配置文件，将被安装到/usr/src/*PACKAGE*-*VERSION* 目录下，一个基本的模板如下

```
PACKAGE_NAME="signelf"
PACKAGE_VERSION="0.0.1"
BUILT_MODULE_NAME[0]="disable_ld_preload"
DEST_MODULE_LOCATION[0]="/extra"
AUTOINSTALL="yes"
```
参数详解：
```
`PACKAGE_NAME` 为dkms模块名称，可以与包名signelf-dkms不一致
`PACKAGE_VERSION` 为内核模块上游版本号
`BUILT_MODULE_NAME[0]` 为由源码编译出的ko文件名称
`DEST_MODULE_LOCATION[0]` 只在当前模块替换树内模块时使用，所以它的值对你来说并不重要，但你需要指定它。
`AUTOINSTALL` 当值为YES时，自动为当前启动的内核安装内核模块
```
更多内容可以查看

**debian/rules:**

添加 `--with dkms` ，dkms addon支持

dh $@ --with dkms


>更多内容请参考dkms的manpage


