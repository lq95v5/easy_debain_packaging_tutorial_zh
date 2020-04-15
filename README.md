# 打包概论
debian 打包是将源码或二进制文件（binary）纳入debian 化的源码树结构中，对源码或者binary进行构建使其可以标准化安装（deb）的一个过程。

简单来说，是通过debian化的源码树构建二进制deb包。

## 打包所需要的工具

- `devscripts` 此包提供了便于DM使用的工具集合，主要使用debuild 和 的dget。debuild 用于在当前环境中打包；dget 用于下载dsc文件
- `pbuilder` 提供了一个chroot净室环境,创建base环境，用于处理dsc文件
- `dh-make` 将源码包归档转换到 Debian 源码包的工具,使用dh_make 创建源码包 tar 包，并添加默认debian模板文件目录
- `dpkg-dev` 这个包提供用于解压缩、构建和上传 Debian 源代码包的开发工具，主要使用dpkg-source生成dsc文件

*其他相关工具*

- `sbuild` 与pbuilder相似，一个净室打包环境
- `debmake` 与dh-make相似
- `cowbuilder`  `cowbuilder` command is a wrapper for pbuilder which allows using pbuilder-like interface over cowdancer environment.


## 基本流程

生成debian源码包 dh_make --createorig &rarr; 编辑源代码下debian目录中控制脚本 &rarr; dpkg-source -b . 生成 dsc文件 &rarr; 使用pbuilder 打包dsc

# 基本环境的配置

## 安装相关工具
```
sudo apt install pbuilder dh-make devscripts debian-archive-keyring
```

## 生成pbuilder打包环境
pbuilder 调用 debootstrap 建立一个基本 Debian 系统（base），并压缩成tgz，构建时会解压tgz，并进入chroot环境打包。通过以下命令构建pbuilder的base环境。
```
sudo pbuilder create --mirror "*镜像仓库*" --distribution "*发行代号*" --basetgz /var/cache/pbuilder/*发行代号*.tgz --allow-untrusted  --debootstrapopts --keyring=/etc/apt/trusted.gpg.d/debian-archive-stretch-stable.gpg  --debootstrapopts --include=debian-archive-keyring,deepin-keyring
```

**参数详解：**
```
--mirror :指定构建仓库地址
--distribution :指定codename发行代号
--basetgz: 指定构建base路径及文件名
--keyring: 指定仓库公钥位置及文件
--include: 添加除base外自定义软件包列表，用','隔开，
```

## 修改pbuilder的base环境

chroot登录base环境，对base环境进行自定义修改

sudo pbuilder --login --save-after-login --basetgz  /path/to/base.tgz

## 添加 hooks 脚本

用于构建前自动执行，刷新base环境软件源缓存

编辑  /var/cache/pbuilder/hooks/D01-apt-get-update

```
#!/bin/bash
apt-get update
```



# 开始打包

## 获取debian源码包的途径与方法
**途径：**

- http://snapshot.debian.org
- https://pkgs.org
- https://launchpad.net

**方法：**

通过dget 命令获取dsc源码包
```
# dget http://pools.corp.deepin.com/deepin/pool/main/w/wget/wget_1.19.5-1.dsc

# ls

wget-1.19.5                  wget_1.19.5-1.dsc        wget_1.19.5.orig.tar.gz.asc
wget_1.19.5-1.debian.tar.xz  wget_1.19.5.orig.tar.gz

```

**通过apt source 命令获取源内源码包**

```
# apt source wget

# ls

wget-1.19.5                  wget_1.19.5-1.dsc        wget_1.19.5.orig.tar.gz.asc
wget_1.19.5-1.debian.tar.xz  wget_1.19.5.orig.tar.gz
```


## 使用pbuilder打包

以打包wget为例

`apt source wget`

`sudo pbuilder --build  --logfile log.txt --basetgz /var/cache/pbuilder/stable.tgz --allow-untrusted --hookdir /var/cache/pbuilder/hooks --use-network yes --aptcache "" --buildresult . *.dsc`

**参数详解：**

```
--logfile: 指定日志文件名及路径
--basetgz: 使用已构建完成的base压缩包
--hookdir: 指定hooks脚本目录
--use-network: 指定是否使用网络
--aptcache:指定cache路径
--buildersult:构建结果路径，.当前路径下
```

## 自行构建源码包

如果需要打包的软件包没有已存在的dsc源码包，需要自行构建dsc。
```
 第一步：获取上游源代码 git svn 等等
 第二步：使用dh_make --createorig产生模板文件debian目录
 第三步：编辑debian目录中的模板文件
 第四步：使用 debuild 在当前环境下构建软件包；或者使用dpkg-source -b . 生成dsc通过pbuilder、sbuild构建
```


## 注意事项&常见问题

- hooks目录需要添加更新软件源的脚本
- pbuilder构建base环境时的秘钥--keyring
- 需要执行dpkg-source -b 刷新debian目录中的模板文件，dh_make生成的源码包要和debian源码树目录内容一致。
- debootstrap需要做发行版本号的软链接处理



# 开始深入阅读

Packaging - Debian Wiki - https://wiki.debian.org/Packaging

Debian Packaging Tutorial - packaging-tutorial.en.pdf - https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.en.pdf

Debian -- 说明文档 - https://www.debian.org/doc/

Debian 维护者指南 - https://www.debian.org/doc/manuals/debmake-doc/index.zh-cn.html

Debian Policy Manual — Debian Policy Manual v4.4.1.1 - https://www.debian.org/doc/debian-policy/index.html


















