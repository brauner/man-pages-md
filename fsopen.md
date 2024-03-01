# NAME

fsopen - Filesystem context creation

# SYNOPSIS

    #include <sys/types.h>
    #include <sys/mount.h>
    #include <unistd.h>
    #include <fcntl.h> /* Definition of AT_* constants */

    int fsopen(const char *fsname, unsigned int flags);

*Note*: There are no glibc wrappers for these system calls.

# DESCRIPTION

**fsopen**() creates a blank filesystem configuration context within the
kernel for the filesystem named in the *fsname* parameter, puts it into
creation mode and attaches it to a file descriptor, which it then
returns. The file descriptor can be marked close-on-exec by setting
**FSOPEN_CLOEXEC** in *flags*.

After calling fsopen(), the file descriptor should be passed to the
**fsconfig**(2) system call, using that to specify the desired
filesystem and security parameters.

When the parameters are all set, the **fsconfig**() system call should
then be called again with **FSCONFIG_CMD_CREATE** as the command
argument to effect the creation.

> **\[!\] NOTE**: Depending on the filesystem type and parameters, this
> may rather share an existing in-kernel filesystem representation
> instead of creating a new one. In such a case, the parameters
> specified may be discarded or may overwrite the parameters set by a
> previous mount - at the filesystem's discretion.

The file descriptor also serves as a channel by which more comprehensive
error, warning and information messages may be retrieved from the kernel
using **read**(2).

Once the creation command has been successfully run on a context, the
context is switched into need-mount mode which prevents further
configuration. At this point, **fsmount**() should be called to create a
mount object.

## Message Retrieval Interface

The context file descriptor may be queried for message strings at any
time by calling **read**(2) on the file descriptor. This will return
formatted messages that are prefixed to indicate their class:

**"e \<message\>"**  
An error message string was logged.

**"i \<message\>"**  
An informational message string was logged.

**"w \<message\>"**  
An warning message string was logged.

Messages are removed from the queue as they're read.

# EXAMPLES

To illustrate the process, here's an example whereby this can be used to
mount an ext4 filesystem on /dev/sdb1 onto /mnt.

    sfd = fsopen("ext4", FSOPEN_CLOEXEC);
    fsconfig(sfd, FSCONFIG_SET_FLAG, "ro", NULL, 0);
    fsconfig(sfd, FSCONFIG_SET_STRING, "source", "/dev/sdb1", 0);
    fsconfig(sfd, FSCONFIG_SET_FLAG, "noatime", NULL, 0);
    fsconfig(sfd, FSCONFIG_SET_FLAG, "acl", NULL, 0);
    fsconfig(sfd, FSCONFIG_SET_FLAG, "user_attr", NULL, 0);
    fsconfig(sfd, FSCONFIG_SET_FLAG, "iversion", NULL, 0);
    fsconfig(sfd, FSCONFIG_CMD_CREATE, NULL, NULL, 0);
    mfd = fsmount(sfd, FSMOUNT_CLOEXEC, MS_RELATIME);
    move_mount(mfd, "", sfd, AT_FDCWD, "/mnt", MOVE_MOUNT_F_EMPTY_PATH);

Here, an ext4 context is created first and attached to sfd. This is then
told where its source will be, given a bunch of options and created.
Then fsmount() is called to create a mount object and **move_mount**(2)
is called to attach it to its intended mountpoint.

And here's an example of mounting from an NFS server and setting a Smack
security module label on it too:

    sfd = fsopen("nfs", 0);
    fsconfig(sfd, FSCONFIG_SET_STRING, "source", "example.com/pub/linux", 0);
    fsconfig(sfd, FSCONFIG_SET_STRING, "nfsvers", "3", 0);
    fsconfig(sfd, FSCONFIG_SET_STRING, "rsize", "65536", 0);
    fsconfig(sfd, FSCONFIG_SET_STRING, "wsize", "65536", 0);
    fsconfig(sfd, FSCONFIG_SET_STRING, "smackfsdef", "foolabel", 0);
    fsconfig(sfd, FSCONFIG_SET_FLAG, "rdma", NULL, 0);
    fsconfig(sfd, FSCONFIG_CMD_CREATE, NULL, NULL, 0);
    mfd = fsmount(sfd, 0, MS_NODEV);
    move_mount(mfd, "", sfd, AT_FDCWD, "/mnt", MOVE_MOUNT_F_EMPTY_PATH);

# RETURN VALUE

On success, both functions return a file descriptor. On error, -1 is
returned, and *errno* is set appropriately.

# ERRORS

The error values given below result from filesystem type independent
errors. Each filesystem type may have its own special errors and its own
special behavior. See the Linux kernel source code for details.

**EFAULT**  
One of the pointer arguments points outside the user address space.

**EINVAL**  
*flags* had an invalid flag set.

**EMFILE**  
The system has too many open files to create more.

**ENFILE**  
The process has too many open files to create more.

**ENODEV**  
Filesystem *fsname* not configured in the kernel.

**ENOMEM**  
The kernel could not allocate sufficient memory to complete the call.

**EPERM**  
The caller does not have the required privileges.

# CONFORMING TO

These functions are Linux-specific and should not be used in programs
intended to be portable.

# VERSIONS

**fsopen**() was added to Linux in kernel 5.1.

# NOTES

Glibc does not (yet) provide a wrapper for the **fsopen**() system
call; call it using **syscall**(2).

# SEE ALSO

**mountpoint**(1), **fsmount**(2), **fsconfig**(2), **fspick**(2),
**move_mount**(2), **open_tree**(2), **umount**(2),
**mount_namespaces**(7), **path_resolution**(7), **findmnt**(8),
**lsblk**(8), **mount**(8), **umount**(8)
