<h3>简介</h3> 
服务器工作时是从网络套接口获取通信的数据，而不是从本地文件或者标准的输入输出。因此，如果要fuzz服务器，则必须将网络数据重定向到标准输入，开源工具Preeny刚好能满足要求，它提供了LD_PRELOAD命令供fuzz爱好者使用。
LD_PRELOAD命令是UNIX的环境变量，它可以影响程序运行时的链接（Runtime linker），它允许用户自定义在程序运行前优先加载的动态链接库。这个功能
主要就是用来选择性的载入不同动态链接库的相同函数。

<h3>Preeny</h3> 
preeny提供了一部分预先写好的函数集，在进行安全研究时可被复写。例如，网络函数connect()和accept()，这些函数部分内容被改写后，便可将网络数据流转换成文件数据流进行操控。Redis使用的协议很简单，在fuzz过程中我们能让AFL执行明确的指令以获取新路径。
<br> preeny安装：
<br>git clone https://github.com/zardus/preeny.git
<br>apt-get install libini-config3 libini-config-dev
<br>cd preeny && make && cd

<h3>fuzzing Redis</h3>
LD_PRELOAD=your_preeny_path/x86_64-linux-gnu/desock.so afl-showmap -m2048 -o/dev/null ./redis-server ~/conf < <(echo "PING");
[*] Executing './redis-server'...

-- Program output begins --
29193:M 20 Sep 18:39:05.230 * Increased maximum number of open files to 10032 (it was originally set to 1024).
             _._                                                  
        _.-``__ ''-._                                             
    _.-``    `.  `_.  ''-._           Redis 3.1.999 (846da5b2/1) 64 bit
.-`` .-```.  ```\/    _.,_ ''-._                                   
(    '      ,       .-`  | `,    )     Running in standalone mode
|`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
|    `-._   `._    /     _.-'    |     PID: 29193
`-._    `-._  `-./  _.-'    _.-'                                   
|`-._`-._    `-.__.-'    _.-'_.-'|                                  
|    `-._`-._        _.-'_.-'    |           http://redis.io        
`-._    `-._`-.__.-'_.-'    _.-'                                   
|`-._`-._    `-.__.-'    _.-'_.-'|                                  
|    `-._`-._        _.-'_.-'    |                                  
`-._    `-._`-.__.-'_.-'    _.-'                                   
    `-._    `-.__.-'    _.-'                                       
        `-._        _.-'                                           
            `-.__.-'                                               

29193:M 20 Sep 18:39:05.232 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
29193:M 20 Sep 18:39:05.232 # Server started, Redis version 3.1.999
29193:M 20 Sep 18:39:05.232 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
29193:M 20 Sep 18:39:05.232 * DB loaded from disk: 0.000 seconds
29193:M 20 Sep 18:39:05.232 * The server is now ready to accept connections on port 6379
-- Program output ends --
[+] Captured 3388 tuples in '/dev/null'.
#
