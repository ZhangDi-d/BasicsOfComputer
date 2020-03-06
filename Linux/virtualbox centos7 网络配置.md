## virtualbox centos7 网络配置

>virtualbox 安装 centos7后,配置网络,权当笔记记录.

[centos7镜像与virtualbox 下载地址](https://blog.csdn.net/ShelleyLittlehero/article/details/104391744) 


### 全局设置
管理->全局设定->网络->勾选NAT网络->如果没有点击新增.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200306135038475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)
### 网络配置:
启用网卡一:仅主机网络,记住其mac地址,后面要使用.

MAC地址为:080027D073EE
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200306134734982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)
启用网卡一:NAT ,记住其mac地址,后面要使用.

MAC地址为:080027252603
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200306134759139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

### 文件修改
1. 启动 centos7;
2. 进入网络配置目录 ,`cd /etc/sysconfig/network-scripts`  ,如果是centos7默认安装的话,应该只有一个ifcfg-enp0s3,可以 `cp ifcfg-enp0s3 ifcfg-enp0s8 `,复制出一个 ifcfg-enp0s8 的文件,并记住修改  ifcfg-enp0s8  其中的 name 和 uuid属性值
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200306135642303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

3. `vim ifcfg-enp0s3` ,编辑 ifcfg-enp0s3 
设置 080027252603  和 dhcp -----> 用来连接外网的;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200306140001557.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

4. `vim ifcfg-enp0s8` ,编辑 ifcfg-enp0s8 
设置 080027D073EE和 static-----> 用来连接主机的;

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200306140246405.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)
5. 修改完成之后 , `service network restart` 重新启动网络服务
6. `ping www.baidu.com` 和 `ping 192.168.56.101`   ,ping百度和ping主机,测试网路连接
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200306140454709.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

7. 在主机命令模式下,`ping 192.168.56.200` ,测试与虚拟机的网络
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200306140556519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

ok


