## 宿主主机如何访问虚拟机中的docker服务

> 网上的回答不一而足,然而都没有解决,最后上了Stack Overflow,找到了答案,国内的小伙伴还得加油呀.

### 环境 
- 宿主机系统 : window 8, 
- 虚拟机软件: Oracle VirtualBox (CentOS7)
- docker version: 19.03.7

### 问题描述 
1. 虚拟机内部`systemctl start docker ` 启动docker , 
2. `docker run -d -p 80:80 nginx ` 启动nginx 服务,
3. 虚拟机ip 192.168.56.200 ,docker 服务ip 172.17.0.16

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311131837437.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

4. 宿主机浏览器` localhost:81` 无法访问nginx .

### 解决方法:
1. 打开virtualbox
2. 选择docker服务所在的虚拟机
3. 点击设置 -> 网络
4. 选择 NAT 网卡
5. 点击高级 -> 端口转发
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031113222255.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

6 添加规则 : tcp 协议 ,主机和子系统端口设置,如 host:80  guest:80 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311132355948.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

7. 浏览器访问 localhost:80 ,可以查看到nginx 界面
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311132531327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)



### 原文:
```
1. Open Oracle VM VirtualBox Manager
2. Select the VM used by Docker
3. Click Settings -> Network
4. Adapter 1 should (default?) be "Attached to: NAT"
5. Click Advanced -> Port Forwarding
6. Add rule: Protocol TCP, Host Port 8080, Guest Port 8080 (leave Host IP and Guest IP empty)
7. Guest is your docker container and Host is your machine
You should now be able to browse to your container via localhost:8080 and your-internal-ip:8080.
```



参考:https://stackoverflow.com/questions/33814696/how-to-connect-to-a-docker-container-from-outside-the-host-same-network-windo