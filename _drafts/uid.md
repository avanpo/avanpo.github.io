---
title: "uid"
categories: linux
---

What happens when an unprivileged user runs the binary

```
-r-s---r-x  1 root root 1024 Mar 17 21:57 binary
```

which executes the following lines of code?

```c
setreuid(geteuid(), geteuid());
system("/bin/sh");
```

You'd expect a privileged shell to open, and that's what happens. But exactly why does that happen?

The POSIX standard defines three different user identifiers. These allow processes to dynamically take on different roles, providing they have the required privileges.

1. The **real uid** identifies the (real) owner of a process. The real uid can be manipulated with [`getuid()`](http://man7.org/linux/man-pages/man2/getuid.2.html) and [`setuid(uid_t uid)`](http://man7.org/linux/man-pages/man2/setuid.2.html).
2. The **effective uid** of a process is used for access control. It is also used as the owner for files created by that process. See [`geteuid(void)`](http://man7.org/linux/man-pages/man2/getuid.2.html) and [`seteuid(uid_t euid)`](http://man7.org/linux/man-pages/man2/seteuid.2.html).
3. The **saved uid** is used to store the effective user id, when the euid is changed. For example, a privileged process may need do some unprivileged work. To do this, it would change its euid. But it would not be able to change back unless its original (privileged) euid was stored somewhere. That's where the saved uid comes in: an unprivileged process may set its euid exclsively to one of uid, suid, or euid.

Processes are managed by the kernel using a process descriptor. In practice, this is a [`task_struct`](https://github.com/torvalds/linux/blob/8f5fd927c3a7576d57248a2d7a0861c3f2795973/include/linux/sched.h#L524) C struct, containing all the process information that the kernel needs. This includes a pointer to a [`cred`](https://github.com/torvalds/linux/blob/8f5fd927c3a7576d57248a2d7a0861c3f2795973/include/linux/cred.h#L111) struct, which stores the different uids mentioned above.

When a process is created, it inherits the real and effective uids of its parent. So when a user runs a binary, the resulting process will typically have the uid and euid of the user. If that were all that was at play, the above example would simply open a shell under the user. So what gives?

It turns out file permissions are also involved. In addition to the familiar read, write and executable bits for owner, group, and others, there are several special modes. One of these is the setuid bit. If this bit is set, executing the file will result in a process with the effective uid of the owner of the file. This is the `s` instead of the `x` in the example above.

When the user executes the binary above, a process is created with the uid of the user, but the euid of root (or 0). The process then sets its own uid to that of its euid (root). It then spawns a shell, which inherits the uid and euid of the process (both root)---i.e. a privileged shell.

So why, then, is the call to `setreuid` even necessary? In theory it shouldn't be, and there are probably some shells for which it isn't. But for security reasons, bash explicitly checks for the setuid case and uses its root powers to drop privileges. In other words, it will drop its effective uid down to its real uid, making the call to `setreuid` a necessity.
