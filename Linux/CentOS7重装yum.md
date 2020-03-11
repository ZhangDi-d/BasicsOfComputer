## CentOS7 重装yum

### 1. 卸载 yum
rpm -aq|grep yum|xargs rpm -e --nodeps

### 2. 下载rpm包
包仓库 :http://mirrors.163.com/centos/7/os/x86_64/Packages/

可以使用wget 方式下载,如下:
```
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-163.el7.centos.noarch.rpm 
```

也可以直接在仓库搜索点击下载,如下 :
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311093106394.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)


下载如下包:

```
python-iniparse-0.4-9.el7.noarch.rpm
python-pycurl-7.19.0-19.el7.x86_64.rpm
python-2.7.5-86.el7.x86_64.rpm
python-urlgrabber-3.10-9.el7.noarch.rpm
python-libs-2.7.5-86.el7.x86_64.rpm

yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
yum-plugin-fastestmirror-1.1.31-52.el7.noarch.rpm
yum-3.4.3-163.el7.centos.noarch.rpm
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031109335813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)


执行 `rpm -ivh python-*`  ,安装python*包;

执行 `rpm -ivh yum-metadata-parser-1.1.4-10.el7.x86_64.rpm  ,安装yum-metadata-parser-1.1.4-10.el7.x86_64.rpm包;

yum-plugin-fastestmirror-1.1.31-52.el7.noarch.rpm 与yum-3.4.3-163.el7.centos.noarch.rpm 相互依赖;

所以执行 `rpm -ivh yum-plugin-fastestmirror-1.1.31-52.el7.noarch.rpm yum-3.4.3-163.el7.centos.noarch.rpm` 安装;


输入 `yum -v`  测试yum 安装;

### 4. 修改yum源

`cd  /etc/yum.repos.d` 进入 yum源目录 ;

从http://mirrors.163.com/.help/CentOS7-Base-163.repo 下载yum源,上传至 /etc/yum.repos.d 目录 ,备份原有repo,

修改CentOS7-Base-163.repo 名称为 CentOS-Base.repo (这一步不是必须的);

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311094034365.png)


修改CentOS-Base.repo 内容 ,替换 $release 为 7 (单签centos版本);替换原有网址为163的网址:

完整内容如下 : 
```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
[base]
name=CentOS-7 - Base - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/7/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-7 - Updates - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=updates
baseurl=http://mirrors.163.com/centos/7/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-7 - Extras - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=extras
baseurl=http://mirrors.163.com/centos/7/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-7 - Plus - 163.com
baseurl=http://mirrors.163.com/centos/7/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

```

### 5. yum update 
运行makecache 生成缓存
```
yum makecache 
```
运行yum clean all
```
yum clean all
```
更新yum文件
```
yum update
```

### yum update 过程中,可能会出现以下问题:

#### 发现 XX 个已存在的 RPM 数据库问题 :

1.首先`yum clean all` ,

2.安装 package-cleanup工具，有下面命令就不需要安装了，有的系统会自带
```
yum install yum-utils
```
然后更新一下仓库
```
package-cleanup --cleandupes
```

####  There are unfinished transactions remaining

1.安装 yum-complete-transaction（这是一个能发现未完成或被中断的yum事务的程序）
```
yum -y install yum-utils
```
2.清除yum缓存
```
yum clean all
```
3.运行 yum-complete-transaction,清理未完成事务
```
yum-complete-transaction --cleanup-only
```

#### conflicts with file from package

conflicts with file from package的问题导致软件安装失败。

需要使用如下命令解决
```
rpm -ivh --replacefiles xxxx.rpm
```


#### Rpmdb checksum is invalid: pkg checksums
RUN rpm --rebuilddb命令可以一条条修复rpm, 但是发现

执行 `yum clean all` , `yum makecache` 更加直接 .














