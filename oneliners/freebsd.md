# DTrace One Liners for FreeBSD

[FreeBSD](http://www.freebsd.org) has supported DTrace since FreeBSD 8
and support was turned on by default for FreeBSD 10.  The following
library of DTrace one-liners were last tested on FreeBSD 10.0 and were
originally imported from the [FreeBSD Wiki](http://wiki.freebsd.org).

## Kernel Locks ##

### Sum kernel adaptive lock block time by process name (ns):
```
dtrace -n 'lockstat:::adaptive-block { @[execname] = sum(arg1); }'
```

### Summarize adaptive lock block time distribution by process name (ns):
```
dtrace -n 'lockstat:::adaptive-block { @[execname] = quantize(arg1); }'
```

### Sum kernel adaptive lock block time by kernel stack trace (ns):
```
dtrace -n 'lockstat:::adaptive-block { @[stack()] = sum(arg1); }'
```

### Sum kernel adaptive lock block time by lock name (ns):
```
dtrace -n 'lockstat:::adaptive-block { @[arg0] = sum(arg1); } END { printa("%40a %@16d ns\n", @); }'
```

### Sum kernel adaptive lock block time by calling function (ns):
```
dtrace -n 'lockstat:::adaptive-block { @[caller] = sum(arg1); } END { printa("%40a %@16d ns\n", @); }'
```

## Namecache ##

### Count namecache lookups by program
```
dtrace -n 'vfs:namecache:lookup: { @missing[execname] = count(); }'
```

### Count namecache misses by program
```
dtrace -n 'vfs:namecache:lookup:miss { @missing[execname] = count(); }'
```

## Raw Kernel Tracing ##

## Count kernel slab memory allocation by function:
```
dtrace -n 'fbt::kmem*:entry { @[probefunc] = count(); }'
```

### Count kernel slab memory allocation by calling function:
```
dtrace -n 'fbt::kmem*:entry { @[caller] = count(); } END { printa("%40a %@16d\n", @); }'
```

### Count kernel malloc() by calling function:
```
dtrace -n 'fbt::malloc:entry { @[caller] = count(); } END { printa("%40a %@16d\n", @); }'
```

### Count kernel malloc() by kernel stack trace:
```
dtrace -n 'fbt::malloc:entry { @[stack()] = count(); }'
```

### Summarize vmem_alloc()s by arena name and size distribution:
```
dtrace -n 'fbt::vmem_alloc:entry { @[args[0]->vm_name] = quantize(arg1); }'
```

### Summarize TCP life span in seconds:
```
dtrace -n 'fbt::tcp_close:entry { @["TCP life span (seconds):"] =
    quantize((uint32_t)(`ticks - args[0]->t_starttime) / `hz); }'
```
