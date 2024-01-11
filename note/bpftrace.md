# Skill
```
bpftrace -lv 'tracepoint:syscalls:sys_enter_read'
```

# One-liners command
```
## Cpu
bpftrace -e 'tracepoint:syscalls:sys_enter_execve { join(args->argv); }'
bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[pid, comm] = count(); }'
bpftrace -e 'profile:hz:49 /pid == 189/ { @[ustack] = count(); }'

## Memory
bpftrace -e tracepoint:syscalls:sys_enter_brk { @[ustack, comm] = count(); }
bpftrace -e 'tracepoint:vmscan:* { @[probe]++; }'

## Filesystem
bpftrace -e 't:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'
bpftrace -e 'tracepoint:syscalls:sys_exit_read { @ = hist(args->ret); }'
bpftrace -e 'kprobe:vfs_* { @[probe] = count(); }'
bpftrace -e 'tracepoint:ext4:* { @[probe] = count(); }'

## Disk
bpftrace -e 't:block:block_rq_issue { @bytes = hist(args->bytes); }'
bpftrace -e 't:block:block_rq_issue { @[ustack] = count(); }'
bpftrace -e 't:block:block_rq_issue { @[args->rwbs] = count(); }'

## Network
bpftrace -e 'kr:tcp_recvmsg /retval >= 0/ { @recv_bytes[comm] = hist(retval); }'
bpftrace -e 't:syscalls:sys_enter_accept* { @[pid, comm] = count(); }'
bpftrace -e 'kr:sock_sendmsg,kr:sock_recvmsg /retval > 0/ { @[pid, comm] = sum(retval); }'
```
