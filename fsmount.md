# NAME

fsmount - Filesystem mount creation

# SYNOPSIS

    #include <sys/types.h>
    #include <sys/mount.h>
    #include <unistd.h>
    #include <fcntl.h> /* Definition of AT_* constants */

    int fsmount(int fd, unsigned int flags, unsigned int mount_attrs);

*Note*: There are no glibc wrappers for these system calls.

# DESCRIPTION

**fsmount**() takes the file descriptor returned by **fsopen**() and
creates a mount object for the filesystem root specified there. The
attributes of the mount object are set from the *mount_attrs* parameter.
The attributes specify the propagation and mount restrictions to be
applied to accesses through this mount.

The mount object is then attached to a new file descriptor that looks
like one created by **open**(2) with **O_PATH** or **open_tree**(2).
This can be passed to **move_mount**(2) to attach the mount object to a
mountpoint, thereby completing the process.

The file descriptor returned by fsmount() is marked close-on-exec if
FSMOUNT_CLOEXEC is specified in *flags*.

After fsmount() has completed, the context created by fsopen() is reset
and moved to reconfiguration state, allowing the new superblock to be
reconfigured. See **fspick**(2) for details.

# RETURN VALUE

On success, **fsmount**(2) returns a file descriptor. On error, -1 is
returned, and *errno* is set appropriately.

# ERRORS

The error values given below result from filesystem type independent
errors. Each filesystem type may have its own special errors and its own
special behavior. See the Linux kernel source code for details.

**EBUSY**  
The context referred to by *fd* is not in the right state to be used by
**fsmount**().

**EFAULT**  
One of the pointer arguments points outside the user address space.

**EINVAL**  
*flags* had an invalid flag set.

**EINVAL**  
*mount_attrs,* includes invalid **MOUNT_ATTR\_\*** flags.

**EMFILE**  
The system has too many open files to create more.

**ENFILE**  
The process has too many open files to create more.

**ENOMEM**  
The kernel could not allocate sufficient memory to complete the call.

# CONFORMING TO

These functions are Linux-specific and should not be used in programs
intended to be portable.

# VERSIONS

**fsmount**() were added to Linux in kernel 5.1.

# NOTES

Glibc does not (yet) provide a wrapper for  the
**fsmount**() system call; call it using **syscall**(2).

# SEE ALSO

**mountpoint**(1), **fsconfig**(2), **fsopen**(2), **fspick**(2),
**move_mount**(2), **open_tree**(2), **umount**(2),
**mount_namespaces**(7), **path_resolution**(7), **findmnt**(8),
**lsblk**(8), **mount**(8), **umount**(8)
