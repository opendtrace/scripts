# DTrace One Liners

A `DTrace` one liner is a simple invocation of the `dtrace(1)` command
that can be used to find out something about the system being traced.
One liners can be general or specific to the underlying operating
system, depending on the provider being used or the data to be
extracted.  One liners that are expected to work across all operating
systems are included in this `README` file while those specific to
various operating systems and software are kept in their own files.

## Finding Probes

### List probes and search for string "foo":
```
dtrace -l | grep foo
```

### Summarize probes by providers:
```
dtrace -l | awk '{ print $2 }' | sort | uniq -c | sort -n
```

### List probes for a particular provider
```
dtrace -l -P syscall
```

### List the arguments for a probe
```
dtrace -lv syscall::write:entry
```

## Syscalls ##

### Trace file opens with process and filename:
```
dtrace -n 'syscall::open*:entry { printf("%s %s", execname, copyinstr(arg0)); }'
```

### Count system calls by program name:
```
dtrace -n 'syscall:::entry { @[execname] = count(); }'
```

### Count system calls by syscall:
```
dtrace -n 'syscall:::entry { @[probefunc] = count(); }'
```

### Count system calls by syscall, for PID 123 only:
```
dtrace -n 'syscall:::entry /pid ## 123/ { @[probefunc] = count(); }'
```

### Count system calls by syscall, for all processes with a specific program name ("nginx"):
```
dtrace -n 'syscall:::entry /execname ## "nginx"/ { @[probefunc] = count(); }'
```

### Count system calls by PID and program name:
```
dtrace -n 'syscall:::entry { @[pid, execname] = count(); }'
```

### Summarize requested read() sizes by program name, as power-of-2 distributions (bytes):
```
dtrace -n 'syscall::read:entry { @[execname] = quantize(arg2); }'
```

### Summarize returned read() sizes by program name, as power-of-2 distributions (bytes or error):
```
dtrace -n 'syscall::read:return { @[execname] = quantize(arg1); }'
```

### Summarize read() latency as a power-of-2 distribution by program name (ns):
```
dtrace -n 'syscall::read:entry { self->ts = timestamp; } syscall::read:return /self->ts/ {
    @[execname, "ns"] = quantize(timestamp - self->ts); self->ts = 0; }'
```

### Summarize read() latency as a linear distribution (0 to 1000, step 5) by program name (ms):
```
dtrace -n 'syscall::read:entry { self->ts = timestamp; } syscall::read:return /self->ts/ {
    @[execname, "ms"] = lquantize((timestamp - self->ts) / 1000000, 0, 1000, 5); self->ts = 0; }'
```

### Summarize read() on-CPU duration as a power-of-2 distribution by program name (ns):
```
dtrace -n 'syscall::read:entry { self->ts = vtimestamp; } syscall::read:return /self->ts/ {
    @[execname, "ns"] = quantize(vtimestamp - self->ts); self->ts = 0; }'
```

### Count read() variants that "nginx" is using (if previous one-liners didn't work):
```
dtrace -n 'syscall::*read*:entry /execname ## "nginx"/ { @[probefunc] = count(); }'
```

### Summarize returned pread() sizes for "nginx" as distributions (bytes or error):
```
dtrace -n 'syscall::pread:return /execname ## "nginx"/ { @ = quantize(arg1); }'
```

### Count socket accept() variants by process name:
```
dtrace -n 'syscall::*accept*:return { @[execname] = count(); }'
```

### Count socket connect() variants by process name:
```
dtrace -n 'syscall::*connect*:return { @[execname] = count(); }'
```

### Summarize returned pread() sizes for "nginx"... and label the output:
```
dtrace -n 'syscall::pread:return /execname ## "nginx"/ { @["rval (bytes)"] = quantize(arg1); }'
```

## Process Tracing ##

### Trace new processes showing program name (and args if available):
```
dtrace -n 'proc:::exec-success { trace(curpsinfo->pr_psargs); }'
```

### Count process-level events:
```
dtrace -n 'proc::: { @[probename] = count(); }'
```


## Profiling ##

### Count sampled thread names on-CPU at 997 Hertz:
```
dtrace -n 'profile-997 { @[stringof(curthread->td_name)] = count(); }'
```

### Count sampled non-idle thread names on-CPU at 997 Hertz:
```
dtrace -n 'profile-997 /!(curthread->td_flags & 0x20)/ { @[stringof(curthread->td_name)] = count(); }'
```

### Count sampled on-CPU kernel stacks at 99 Hertz:
```
dtrace -n 'profile-99 /arg0/ { @[stack()] = count(); }'
```

### Count sampled process names and on-CP user stacks at 99 Hertz:
```
dtrace -n 'profile-99 /arg1/ { @[execname, ustack()] = count(); }'
```

## IP ##

### Count IP-level events:
```
dtrace -n 'ip::: { @[probename] = count(); }'
```

## UDP ##

### Count UDP-level events:
```
dtrace -n 'udp::: { @[probename] = count(); }'
```

## TCP ##

### Count TCP-level events:
```
dtrace -n 'tcp::: { @[probename] = count(); }'
```

### Trace TCP accepted connections by remote IP address:
```
dtrace -n 'tcp:::accept-established { trace(args[3]->tcps_raddr); }'
```

### Count TCP passive opens by remote IP address:
```
dtrace -n 'tcp:::accept-established { @[args[3]->tcps_raddr] = count(); }'
```

### Count TCP active opens by remote IP address:
```
dtrace -n 'tcp:::connect-established { @[args[3]->tcps_raddr] = count(); }'
```

### Count TCP sent messages by remote IP address:
```
dtrace -n 'tcp:::send { @[args[2]->ip_daddr] = count(); }'
```

### Count TCP received messages by remote IP address:
```
dtrace -n 'tcp:::receive { @[args[2]->ip_saddr] = count(); }'
```

### Summarize TCP sent messages by IP payload size, as a power-of-2 distribution:
```
dtrace -n 'tcp:::send { @[args[2]->ip_daddr] = quantize(args[2]->ip_plength); }'
```



