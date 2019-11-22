linux自动清理30天之前的文件

### 1.删除文件命令：
```
find 对应目录 -mtime +天数 -name "文件名" -exec rm -rf {} \;
```

实例命令：
```
find /home/weblogic/rc-server-tomcat-8081/logs -mtime +30 -name "*.txt" -exec rm -rf {} \;
```

说明：

将/home/weblogic/rc-server-tomcat-8081/logs目录下所有30天前带".txt"的文件删除。具体参数说明如下：

find：linux的查找命令，用户查找指定条件的文件；
```
/home/weblogic/rc-server-tomcat-8081/logs：想要进行清理的任意目录；
```

-mtime：标准语句写法；

+30：查找30天前的文件，这里用数字代表天数；

"*.log"：希望查找的数据类型，"*.jpg"表示查找扩展名为jpg的所有文件，"*"表示查找所有文件，这个可以灵活运用，举一反三；

(签购单接口性能测试时会产生大量签购单图片，并保持在对应的目录下，导致磁盘经常被占满，自动清除前一天签购单图片，脚本中，清除的格式改为.jpg就行)

-exec：固定写法；

rm -rf：强制删除文件，包括目录；

{} \; ：固定写法，一对大括号+空格+\+; 


### 2.计划任务：

若嫌每次手动执行语句太麻烦，可以将这小语句写到一个可执行shell脚本文件中，再设置cron调度执行，那就可以让系统自动去清理相关文件。

#### 2.1 创建shell：
```
touch /home/weblogic/luojie/script/auto-del-30-days-txt.sh

chmod  777 auto-del-30-days-txt.sh
```

新建一个可执行文件auto-del-30-days-txt.sh，并分配可运行权限

#### 2.2 编辑shell脚本：
```
vi auto-del-30-days-txt.sh
```
编辑auto-del-30-days-txt.sh文件如下：
```
#!/bin/sh
find /home/weblogic/rc-server-tomcat-8081/logs -mtime +30 -name "*.txt" -exec rm -rf {} \;
```
ok，保存退出(:wq!)。

#### 2.3计划任务：

键入 crontab  -e 编辑crontab服务文件

将auto-del-30-days-txt.sh执行脚本加入到系统计划任务，到点自动执行

输入：
```
10 0 * * * /home/weblogic/luojie/script/auto-del-30-days-txt.sh >/dev/null 2>&1
```
这里的设置是每天凌晨0点10分执行auto-del-30-days-txt.sh文件进行数据清理任务了。


### 3 定时任务(不借助脚本)
新建定时任务
```
0 0 * * * cd /home/weblogic/rc-server-tomcat-8081/logs && find . -ctime +30 -type f | xargs rm -rf>/dev/null 2>&1
```

查看该用户下的crontab服务是否创建成功， 用 crontab  -l 命令 

启动crontab服务

一般启动服务用  /sbin/service crond start 若是根用户的cron服务可以用 sudo service crond start， 这里还是要注意  下 不同版本linux系统启动的服务的命令也不同 ，像我的虚拟机里只需用 sudo service cron restart 即可，若是在根用下直接键入service cron start就能启动服务

查看服务是否已经运行用 
```
ps -ax | grep cron |grep -v grep |awk '{print $2}'
```
 

 

#### crontab命令

    cron服务提供crontab命令来设定cron服务的，以下是这个命令的一些参数与说明:

    - crontab -u //设定某个用户的cron服务，一般root用户在执行这个命令的时候需要此参数  
　　- crontab -l //列出某个用户cron服务的详细内容
　　- crontab -r //删除没个用户的cron服务
　　- crontab -e //编辑某个用户的cron服务
　　
比如说root查看自己的cron设置:crontab -u root -l

再例如，root想删除fred的cron设置:crontab -u fred -r
　　
在编辑cron服务时，编辑的内容有一些格式和约定，输入:crontab -u root -e
　　
进入vi编辑模式，编辑的内容一定要符合下面的格式:*/1 * * * * ls >> /tmp/ls.txt
任务调度的crond常驻命令crond 是linux用来定期执行程序的命令。当安装完成操作系统之后，默认便会启动此  
任务调度命令。crond命令每分锺会定期检查是否有要执行的工作，如果有要执行的工作便会自动执行该工作。

 

#### crontab命令选项:

     -u指定一个用户

     -l列出某个用户的任务计划

     -r删除某个用户的任务

     -e编辑某个用户的任务

#### cron文件语法:

      分     小时    日       月       星期     命令

      0-59   0-23   1-31   1-12     0-6     command     (取值范围,0表示周日一般一行对应一个任务)

     记住几个特殊符号的含义:

         “*”代表取值范围内的数字,
         “/”代表”每”,
         “-”代表从某个数字到某个数字,
         “,”分开几个离散的数字

#### 任务调度设置文件的写法
      可用crontab -e命令来编辑,编辑的是/var/spool/cron下对应用户的cron文件,也可以直接修改/etc/crontab文件
     具体格式如下：
	 ```
      Minute Hour Day Month Dayofweek   command
      分钟     小时   天     月     天每星期       命令
	 ```
     每个字段代表的含义如下：
     Minute             每个小时的第几分钟执行该任务
     Hour               每天的第几个小时执行该任务
     Day                 每月的第几天执行该任务
     Month             每年的第几个月执行该任务
     DayOfWeek     每周的第几天执行该任务
     Command       指定要执行的程序
     在这些字段里，除了“Command”是每次都必须指定的字段以外，其它字段皆为可选

    字段，可视需要决定。对于不指定的字段，要用“*”来填补其位置。
    举例如下：   

```
5      *       *         *     *     ls             指定每小时的第5分钟执行一次ls命令
30     5       *         *     *     ls             指定每天的 5:30 执行ls命令 
30     7       8         *     *     ls             指定每月8号的7：30分执行ls命令
30     5       8         6     *     ls             指定每年的6月8日5：30执行ls命令 
30     6       *         *     0     ls             指定每星期日的6:30执行ls命令[注：0表示星期天，1表示星期1， 以此类推，
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　也可以用英文来表示，sun表示星期天，mon表示星期一等。]
30     3       10,20     *     *     ls             每月10号及20号的3：30执行ls命令[注：“，”用来连接多个不连续的时段]
25     8-11    *         *     *     ls             每天8-11点的第25分钟执行ls命令[注：“-”用来连接连续的时段]
*/15   *       *         *     *     ls             每15分钟执行一次ls命令 [即每个小时的第0 15 30 45 60分钟执行ls命令 ]
30     6       */10      *     *     ls             每个月中，每隔10天6:30执行一次ls命令[即每月的1、11、21、31日是的6：30执行一次ls 命令。 ]
                                              每天7：50以root 身份执行/etc/cron.daily目录中的所有可执行文件
50     7       *         *     *     root     run-parts     /etc/cron.daily   [ 注：run-parts参数表示，执行后面目录中的所有可执行文件。 ]
```
 

#### 新增调度任务

     新增调度任务可用两种方法：
       1)、在命令行输入: crontab -e 然后添加相应的任务，wq存盘退出。
       2)、直接编辑/etc/crontab 文件，即vi /etc/crontab，添加相应的任务。

#### 查看调度任务
        crontab -l //列出当前的所有调度任务
        crontab -l -u jp   //列出用户jp的所有调度任务

#### 删除任务调度工作
         crontab -r   //删除所有任务调度工作

#### 任务调度执行结果的转向
例1：每天5：30执行ls命令，并把结果输出到/jp/test文件中
	30 5 * * * ls >/jp/test 2>&1
	注：2>&1 表示执行结果及错误信息。
编辑/etc/crontab 文件配置cron  

cron服务每分钟不仅要读一次/var/spool/cron内的所有文件，还需要读一次 /etc/crontab,因此我们配置这个文件也能运用cron服务做一些事情。用crontab配置是针对某个用户的，而编辑/etc/crontab是针对系统的任务。此文件的文件格式是:  

　　SHELL=/bin/bash  

　　PATH=/sbin:/bin:/usr/sbin:/usr/bin 

　　MAILTO=root //如果出现错误，或者有数据输出，数据作为邮件发给这个帐号  

　　HOME=/ //使用者运行的路径,这里是根目录  
　　# run-parts  
```
01   *   *   *   *     root run-parts /etc/cron.hourly         //每小时执行/etc/cron.hourly内的脚本  
02   4   *   *   *     root run-parts /etc/cron.daily           //每天执行/etc/cron.daily内的脚本  
22   4   *   *   0     root run-parts /etc/cron.weekly       　　//每星期执行 /etc/cron.weekly内的脚本  
42   4   1   *   *     root run-parts /etc/cron.monthly     　　//每月去执行/etc/cron.monthly内的脚本  
```

　 大家注意”run-parts”这个参数了，如果去掉这个参数的话，后面就可以写要运行的某个脚本名，而不是文件夹名了

例如：
 1) 在命令行输入: crontab -e 然后添加相应的任务，wq存盘退出。

 2)直接编辑/etc/crontab 文件，即vi /etc/crontab，添加相应的任务
```
11 2 21 10 * rm -rf /mnt/fb 
```