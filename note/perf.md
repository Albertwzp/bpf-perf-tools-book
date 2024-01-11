```
perf list [pre]
perf record -F 99 -a -g -- sleep 10
perf record -e sched:sched_switch/max-stack=5/,sched:sched_wakeup/max-stack=1/ \ -a -- sleep 1
perf report --[stdio|tui] -i file
perf script [--insn-trace --xed] --header -F comm,pid,tid,cpu,time,event,ip,sym,dso,trace
```
# Hardware
```
perf record [-F hz] -vve cycles -a sleep 1
sysctl kernel.perf_event_max_sample_rate
sysctl kernel.perf_cpu_time_max_percent
```
# Software
```
perf record -vve context-switches -a -- sleep 1
```
# Tracepoint
```
perf record -vve sched:sched_switch -a sleep 1
perf record -e block:block_rq_issue -a sleep 10; perf script
cat /sys/kernel/debug/tracing/events/block/block_rq_issue/format
perf record -e block:block_rq_issue --filter 'bytes > 65536' -a sleep 10
```
# Probe
```
## kprobes
perf probe --add do_nanosleep
#perf probe --add 'do_nanosleep%return [$retval]'
perf probe --vars do_nanosleep
perf probe -[nv] 'do_nanosleep mode[=%si:x32]'
perf record -e probe:do_nanosleep -a sleep 5
perf script
perf probe --del do_nanosleep

## uprobes
perf probe -x /lib/x86_64-linux-gnu/libc.so.6 --add fopen[%return $retval]
perf probe -x /lib/x86_64-linux-gnu/libc.so.6 --add 'fopen filename=+0(%di):string mode=%si:u8'
perf probe -x /lib/x86_64-linux-gnu/libc.so.6 --vars fopen
perf probe --del probe_libc:fopen

## USDT
perf build-cache --add $(which python3)
perf probe  <PROBE> [OP]
perf record -e <PROBE> -aR sleep 10
```

# State
```
perf stat -e sched:sched_switch -a -- sleep 10
perf stat -e 'sched:*,block:*' -a --sleep 10
perf stat -e sched:sched_stat_runtime -[a|A] --filter 'prev_pid == 1' -I 1000
perf stat -e cycles,instructions -a -- sleep 10
```

# Trace
```
perf trace -e block:block_rq_issue,block:block_rq_complete
perf trace -e syscalls:*enter_mmap --filter='flags==SHARED'
perf trace -e block:block_rq_issue,block:block_rq_complete --no-syscalls
```
