# Ftrace
```
trace-cmd list [-f]
cat /sys/kernel/debug/tracing/available_tracers
trace-cmd record -p [function|function_graph] -P 25314
trace-cmd record -e sched:sched_switch
```
