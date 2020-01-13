## Linux 守护进程的启动方法

>"守护进程"（daemon）就是一直在后台运行的进程（daemon）。

本文介绍如何将一个 Web 应用，启动为守护进程。

### 一、问题的由来

Web应用写好后，下一件事就是启动，让它一直在后台运行。

这并不容易。举例来说，下面是一个最简单的Node应用server.js，只有6行。

```
var http = require('http');

http.createServer(function(req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World');
}).listen(5000);
```

你在命令行下启动它。

```
$ node server.js
```
看上去一切正常，所有人都能快乐地访问 5000 端口了。但是，一旦你退出命令行窗口，这个应用就一起退出了，无法访问了。

怎么才能让它变成系统的守护进程（daemon），成为一种服务（service），一直在那里运行呢？

### 二、前台任务与后台任务
上面这样启动的脚本，称为"前台任务"（foreground job）。它会独占命令行窗口，只有运行完了或者手动中止，才能执行其他命令。

变成守护进程的第一步，就是把它改成"后台任务"（background job）。

```
$ node server.js &
```
只要在命令的尾部加上符号&，启动的进程就会成为"后台任务"。如果要让正在运行的"前台任务"变为"后台任务"，可以先按ctrl + z，然后执行bg命令（让最近一个暂停的"后台任务"继续执行）。

"后台任务"有两个特点:

1. 继承当前 session （对话）的标准输出（stdout）和标准错误（stderr）。因此，后台任务的所有输出依然会同步地在命令行下显示。
2. 不再继承当前 session 的标准输入（stdin）。你无法向这个任务输入指令了。如果它试图读取标准输入，就会暂停执行（halt）。

可以看到，"后台任务"与"前台任务"的本质区别只有一个：是否继承标准输入。所以，执行后台任务的同时，用户还可以输入其他命令。

### 三、SIGHUP信号
变为"后台任务"后，一个进程是否就成为了守护进程呢？或者说，用户退出 session 以后，"后台任务"是否还会继续执行？

Linux系统是这样设计的。
```
1.用户准备退出 session
2.系统向该 session 发出SIGHUP信号
3.session 将SIGHUP信号发给所有子进程
4.子进程收到SIGHUP信号后，自动退出

```
上面的流程解释了，为什么"前台任务"会随着 session 的退出而退出：因为它收到了SIGHUP信号。

那么，"后台任务"是否也会收到SIGHUP信号？

这由 Shell 的huponexit参数决定的。

```
$ shopt | grep huponexit
```

执行上面的命令，就会看到huponexit参数的值。

大多数Linux系统，这个参数默认关闭（off）。因此，session 退出的时候，不会把SIGHUP信号发给"后台任务"。所以，一般来说，"后台任务"不会随着 session 一起退出。

### 四、disown 命令
通过"后台任务"启动"守护进程"并不保险，因为有的系统的huponexit参数可能是打开的（on）。

更保险的方法是使用disown命令。它可以将指定任务从"后台任务"列表（jobs命令的返回结果）之中移除。一个"后台任务"只要不在这个列表之中，session 就肯定不会向它发出SIGHUP信号。


$ node server.js &
$ disown
执行上面的命令以后，server.js进程就被移出了"后台任务"列表。你可以执行jobs命令验证，输出结果里面，不会有这个进程。

disown的用法如下。
```

# 移出最近一个正在执行的后台任务
$ disown

# 移出所有正在执行的后台任务
$ disown -r

# 移出所有后台任务
$ disown -a

# 不移出后台任务，但是让它们不会收到SIGHUP信号
$ disown -h

# 根据jobId，移出指定的后台任务
$ disown %2
$ disown -h %2
```


### 五、标准 I/O
使用disown命令之后，还有一个问题。那就是，退出 session 以后，如果后台进程与标准I/O有交互，它还是会挂掉。

还是以上面的脚本为例，现在加入一行。

```
var http = require('http');

http.createServer(function(req, res) {
  console.log('server starts...'); // 加入此行
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World');
}).listen(5000);
```
启动上面的脚本，然后再执行disown命令。

```
$ node server.js &
$ disown
```
接着，你退出 session，访问5000端口，就会发现连不上。

这是因为"后台任务"的标准 I/O 继承自当前 session，disown命令并没有改变这一点。一旦"后台任务"读写标准 I/O，就会发现它已经不存在了，所以就报错终止执行。

为了解决这个问题，需要对"后台任务"的标准 I/O 进行重定向。

```
$ node server.js > stdout.txt 2> stderr.txt < /dev/null &
$ disown
```
上面这样执行，基本上就没有问题了。

### 六、nohup 命令
还有比disown更方便的命令，就是nohup。

```
$ nohup node server.js &
```
nohup命令对server.js进程做了三件事。

```
1. 阻止SIGHUP信号发到这个进程。
2. 关闭标准输入。该进程不再能够接收任何输入，即使运行在前台。
3. 重定向标准输出和标准错误到文件nohup.out。

```
也就是说，nohup命令实际上将子进程与它所在的 session 分离了。

注意，nohup命令不会自动把进程变为"后台任务"，所以必须加上&符号。



### Linux 常用SIG信号及其键值
01 SIGHUP 挂起（hangup）
02 SIGINT 中断，当用户从键盘按^c键或^break键时
03 SIGQUIT 退出，当用户从键盘按quit键时
04 SIGILL 非法指令
05 SIGTRAP 跟踪陷阱（trace trap），启动进程，跟踪代码的执行
06 SIGIOT IOT指令
07 SIGEMT EMT指令
08 SIGFPE 浮点运算溢出
09 SIGKILL 杀死、终止进程 
10 SIGBUS 总线错误
11 SIGSEGV 段违例（segmentation  violation），进程试图去访问其虚地址空间以外的位置
12 SIGSYS 系统调用中参数错，如系统调用号非法
13 SIGPIPE 向某个非读管道中写入数据
14 SIGALRM 闹钟。当某进程希望在某时间后接收信号时发此信号
15 SIGTERM 软件终止（software  termination）
16 SIGUSR1 用户自定义信号1
17 SIGUSR2 用户自定义信号2
18 SIGCLD 某个子进程死
19 SIGPWR 电源故障


----------------------------------

转载自：
https://blog.csdn.net/God2469/article/details/9713395

