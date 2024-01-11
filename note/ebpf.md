# 概览
tracing, sampling, observeability
1. 早期前端支持工具为BCC(libbcc,libbpf),适合复杂脚本和后台进程开发,可结合其他语言使用.
2. 现在出现基于libbcc和libbpf的bpftrace,适用单行命令和简单脚本.
3. ply适用于嵌入式环境,支持bpftrace语法格式转化，后续直接支持bpftrace语法格式.
传统工具提供性能分析的起点，bpf跟踪工具可提供更加深入的调查.
(动态插桩:存在接口稳定性问题,编译器优化函数内联)Debugger(1990) -> Dprobe(2000) -> kprobe(2004) -> DTrace(2005) -> uprobe(2012)
(静态插桩)事件名编码到软件中，数量非常有限，需要额外维护
```
bpftrace -e 'tracepoint:syscalls:sys_enter_open { printf("%s %s\n", comm, str(args->filename)); }'
bpftrace -e 'kprobe:vfs_read { @[comm] = count(); }'
bpftrace -v <SCRIPT>
opensnoop -x
```

# 技术背景
```
bpftool prog load <hello.bpf.o> </sys/fs/bpf/hello>
bpftool net attach xdp id <ID> dev <eth0>
ip link set dev eth0 xdp obj hello.bpf.o sec xdp
ip link set dev eth0 xdp off

bpftool prog show [id|name|tag|pinned] <P>
bpftool prog dump xlated id <ID> [opcode|visual|linum]
bpftool prog dump jited id <ID> 
bpftool map list
bpftool map dump [name|id] <P>
bpftool btf dump [name|id] <P> 
```

# 性能分析
## 指标: 延迟，速率，吞吐量，利用率，成本.
明确目标，具体针对哪个指标(如果对路径所有事件跟踪则工具会绑定应用且带来额外开销，推荐小而专的工具.
不通阶段的性能分析活动通常对应不同的BPF性能分析工具.
多重性能问题，通常先找出影响最重要的那个，可以找关注的项目查看bug列表搜索性能相关.
## 性能分析方法论
业务负载画像：谁产生,为什么产生，组成，随时间变化
下钻分析,逐层拆分问题：1.高层开始分析2.检查下一层细节3.挑选感兴趣线索4.如果问题未解决回第二步
USE(使用率，饱和度，错误率):从重要的问题出发，而不管是否已有工具方便测量
检查清单法：利用一系列工具和指标对照或检查用于指导公司各层次实施操作，笼统无逻辑.
Linux 60秒分析：uptime, demsg|tail, vmstat 1, mpstat -p ALL 1, pidstat 1, iostat -xz 1, free -w, sar -n DEV 1, sar -n TCP,ETCP 1, top
BCC 检查：execsnoop, opensnoop, ext4slower, biolatency -m, biosnoop, cachestat, tcpconnect, tcpaccept, tcpretrans, runqlat, profile

# BCC
bcc-tools bcc
```
# Debian install bcc
echo deb http://cloudfront.debian.net/debian sid main >> /etc/apt/sources.list
sudo apt-get install -y bpfcc-tools libbpfcc libbpfcc-dev linux-headers-$(uname -r)
[bpf]<https://github.com/iovisor/bcc/blob/master/INSTALL.md>

funccount [tcp_drop|'vfs_*'|-i 1 c:pthread_mutex_lock|'t:syscalls:sys_enter_*']

stackcount -f -P -D 10 ktime_get >out.txt
flamegraph.ph --hash --bgcolors=gray < ./out.txt >out.svg

trace 'r::do_sys_open "ret: %d", retval'
trace -tKU 'r::sock_alloc "open %llx", retval' '__sock_release "close %llx", arg1'

argdist -H 'r::__tcp_select_window():int:$retval'

b=BPF(text=bpf_text, debug=XXX)
bpflist -vv
# 在4.17之前的内核上进行Ftrace状态清理来移除激活事件源,之后的内核PMC通过perf_event_open通过崩溃后自动文件描述符
reset-trace -v
```

# bpftrace
```
bpftrace -e 'kretprobe:vfs_read { @bytes=hist(retval); }'
bpftrace -e 't:block:block_rq_insert {printf("Block I/) by %s\n", kstack);}'
bpftrace -e 'tracepoint:syscalls:sys_enter_execve { join(args->argv);}'
bpftrace -e 'ur:/bin/bash:readline {printf("%s\n", str(retval));}'
bpftrace -e 't:block:block_rq_insert {@[kstack(3), comm] = count();}'
bpftrace -e 'k:do_nanosleep{printf("%s", ustack(perf));}'
bpftrace -e 'tracepoint:timer:hrtimer_start {@[ksym(args->function)]=count();} interval:s:5 {exit();}'
bpftrace -e 'uprobe:/bin/bash:readline { printf("PS1: %s\n", str(*uaddr("ps1_prompt")));}'
bpftrace --unsafe -e 'tracepoint:syscalls:sys_enter_nanosleep { system("ps -p %d\n", pid);}'
```
perf,Ftrace,SystemTap,LTTng,ApplicationBound...
kernel feature: CONFIG_BPF=y,CONFIG_BPF_SYSCALL=y,CONFIG_BPF_JIT=y,CONFIG_HAVE_EBPF_JIT=y,CONFIG_BPF_EVENTS=y

# CPU
传统工具
```
mpstat -P ALL 1
tlbstat -C0 1
perf stat -d <COMMAND>
perf record -F 99 -a -g -- sleep 10
perf report -n --stdio
perf script --header | ./stackcollapse-perf.pl | ./flamegraph.pl >flamel.svg
perf stat -e sched:sched_process_exec -I 1000	#exec系统调用频率
perf record -e sched:sched_process_exec -a	#详细调用命令
perf sched record -- sleep 1
perf sched timehist
```
bpf 工具
```
execsnoop(BCC/BT)
exitsnoop(BCC)
runqlat(BCC/BT) -m -P --pidnss -p <PID> -T <TIME> <COUNT>
runqlen(BCC/BT) <TIME> <COUNT>
runqslower(BCC) -p <PID>
cpudist(BCC)
profile -af 30 >out.stacks
git clone https://github.com/brendangregg/FlameGraph.git
./flamegraph.pl --color=java < ../out.stacks >out.svg
bpftrace -e 'profile:hz:49 /pid/ { @samples[ustack, kstack, comm] = count(); }'
```

# 
# Mem
传统工具
```
pmap -x <PID>
sar -B 1
perf stat -e LLC-loads,LLC-load-misses -a -I 1000
perf record -e L1-dcache-load-misses -a -C 100000
oomkill(BCC/BT)
memleak -p <PID>
stackcount t:exceptions:page_fault_kernel
```

# Network
```
bpftrace -l 't:tcp:*'
``
