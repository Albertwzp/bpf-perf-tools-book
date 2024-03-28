# Ftrace

## Filesystem
```
mount -t debugfs,tracefs
cd /sys/kernel/debug/tracing
```
## Function profiler
```
cat /sys/kernel/debug/tracing/available_tracers
cat /sys/kernel/debug/tracing/set_ftrace_filter
echo 1 >/sys/kernel/debug/tracing/function_profile_enabled
head trace_stat/function*
```

## Function tracing
```
echo 1 >tracing_on
echo '*' >set_ftrace_filter
echo function >current_tracer
cat trace >/tmp/out.txt

cat trace_pipe >/tmp/pipe_out.txt
cat options/*
```
## Tracepoints
```
echo 1 >events/block/block_rq_issue/enable
echo 'bytes > 65536' > events/block/block_rq_insert/filter #echo 0 to resets filter
echo 'traceoff if bytes > 65536' > events/block/block_rq_insert/trigger
cat trace_pipe

```
## kprobes
```
echo 'p:brendan do_nanosleep hrtimer_sleeper=$arg1 hrtimer_mode=$arg2' >> kprobe_events
echo 'r:xrendan do_nanosleep ret=$retval' >> kprobe_events
echo 1 > events/kprobes/brendan/enable
cat trace_pipe
echo '-:brendan' >>kprobe_events
echo 'hrtimer_mode != 1' > events/kprobes/brendan/filter
cat events/kprobes/brendan/format
cat /sys/kernel/debug/tracing/kprobe_profile
```
## uprobes
```
readelf -s /bin/bash |grep -2 readline
echo 'p:urendan /bin/bash:0xb61e0' >> uprobe_events
echo 1 >events/uprobes/brendan/enable
cat trace_pipe
echo 0 > events/uprobes/brendan/enable
echo '-:urendan' >> uprobe_events
cat /sys/kernel/debug/tracing/uprobe_profile
```
## Ftrace function_graph
```
echo do_nanosleep >set_graph_function
echo function_graph >current_tracer
echo do_nanosleep >set_ftrace_filter
```
## Ftrace hwlat
```
echo hwlat >current_tracer
```
## Ftrace Hist Triggers
```
echo 'hist:key=common_pid,id' > events/raw_syscalls/sys_enter/trigger 
cat events/raw_syacalls/sys_enter/hist
echo '!hist:key=common_pid' > events/raw_syscalls/sys_enter/trigger
cat events/raw_syscalls/sys_enter/format
echo 'hist:key=id.syscall if common_pid==32396' > events/raw_syscalls/sys_enter/trigger

echo 'hist:key=stacktrace' > events/block/block_rq_issue/trigger
cat events/block/block_rq_issue/hist

echo 'syscall_latency u64 lat_us; long id' >> synthetic_events
echo 'hist:keys=common_pid:ts0=common_timestamp.usecs' >> events/raw_syscalls/sys_enter/trigger
echo 'hist:keys=common_pid:lat_us=common_timestamp.usecs-$ts0:' 'onmatch(raw_syscalls.sys_enter).trace(syscall_latency,$lat_us,id)' >>events/raw_syscalls/sys_exit/trigger
echo 'hist:keys=lat_us,id.syscall:sort=lat_us' >> events/synthetic/syscall_latency/trigger
cat events/synthetic/syscall_latency/hist
```
## Ftrace command
```
apt install trace-cmd
trace-cmd list [-f]
trace-cmd record -p [function|function_graph] -P 25314
trace-cmd record -e sched:sched_switch
trace-cmd listen -p 10086
trace-cmd record ... -N 127.0.0.1:10086
trace-cmd report --cpu 0
```
