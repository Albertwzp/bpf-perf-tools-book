## 内核态线程使用[xxx]标识
## load avg
```
make VMLINUX_BTF=/sys/kernel/btf/vmlinux -M samples/bpf
```
```
grep -c 'model name' /proc/cpuinfo		#核心数
```
## net
```
cat /sys/class/net/*/queues/rx-*/rps_cpus
perf top -C 0 -U
funcgraph-perf -m 1 -a -d 6 estimation_timer
```
## build vmlinux.h
```
bpftool btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h
```
## disassmble
```
llvm-objdump -S hello.bpf.o
```
## strace bpf syscall
```
strace -e bpf ./hello.py
strace -e bpf,perf_event_open,ioctl,ppoll ./hello-buffer-config.py
```
## kubectl-trace
```
kubectl-trace -n <NS> run pod/<POD> -e 'uretprobe:/proc/$container_pid/exe:main { printf("exit: %d\n", retval) }'
```
