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
1.PING 测试
<br>LD_PRELOAD=your_preeny_path/x86_64-linux-gnu/desock.so afl-showmap -m2048 -o/dev/null ./redis-server ~/conf < <(echo "PING");
2.创建Redis命令目录供AFL-fuzz使用
<br># mkdir testcases syncdir dictionary && cd dictionary
<br># for i in `curl https://raw.githubusercontent.com/antirez/redis/unstable/src/server.c | grep Command, | sed 's/ //g' | grep -oP <br>'{"(.*?)"' | sort | uniq | sed -e s/\"//g -e s/{//g`; do echo $i> `uuid`; done
<br># cd
3.AFL-fuzz
# LD_PRELOAD=~/preeny/x86_64-linux-gnu/desock.so afl-fuzz -i ~/testcases/ -o ~/syncdir/ -x ~/dictionary/  -m2048 ./redis-server ./conf
