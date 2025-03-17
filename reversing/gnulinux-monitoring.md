# GNU/Linux Monitoring

Here there are some examples of utilities useful for tracing and monitoring
programs in GNU/Linux.


## Find files opened by a process

Files are opened by executing some syscall of the `open` family, commonly
`openat`, so we can monitor which files are being opening by a process with
[strace](https://www.man7.org/linux/man-pages/man1/strace.1.html). The `-e file` will let us see all the syscalls related to
files. Here is a silly example with `cat`:

```shell
$ strace -e file cat /etc/hostname 
execve("/usr/bin/cat", ["cat", "/etc/hostname"], 0x7ffed16c3ad8 /* 55 vars */) = 0
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/etc/hostname", O_RDONLY) = 3
snake
+++ exited with 0 +++
```

## Find which processes open a file

If we want to discover which process is opening an specific file, we can monitor
it with [opensnoop](https://manpages.org/opensnoop/8) from the [bpfcc-tools](https://github.com/iovisor/bcc). The bad thing about
`opensnoop` is that doesn't allow to filter by file path, but `grep` can do the
job. Here is an example:

```shell
$ sudo opensnoop-bpfcc -TeU | grep /etc/hosts
6.956550000   1000  11842  vim                 3   0 00000000 /etc/hosts
6.956568000   1000  11842  vim                 3   0 00000000 /etc/hosts
46.206869000  1000  2122   DNS Res~ver #16    88   0 02000000 /etc/hosts
46.670103000  1000  2122   DNS Res~ver #16   146   0 02000000 /etc/hosts
```

As we can see the `/etc/hosts` file was opened by vim and some DNS process (as
it is expected).


## References

- [strace](https://www.man7.org/linux/man-pages/man1/strace.1.html)
- [bpfcc-tools](https://github.com/iovisor/bcc)
