# NAME

fspick - Select filesystem for reconfiguration

# SYNOPSIS

    #include <sys/types.h>


    #include <sys/mount.h>


    #include <unistd.h>


    #include <fcntl.h> /* Definition of AT_* constants */

    int fspick(int dirfd, const char *pathname, unsigned int flags);

*Note*: There is no glibc wrapper for this system call.

# DESCRIPTION

**fspick**() creates a new filesystem configuration context within the
kernel and attaches a pre-existing superblock to it so that it can be
reconfigured (similar to **mount**(8) with the "-o remount" option). The
configuration context is marked as being in reconfiguration mode and
attached to a file descriptor, which is returned to the caller. This can
be marked close-on-exec by setting **FSPICK_CLOEXEC** in *flags*.

The target is whichever superblock backs the object determined by *dfd*,
*pathname* and *flags*. The following can be set in *flags* to control
the pathwalk to that object:

**FSPICK_SYMLINK_NOFOLLOW**  
Don't follow symbolic links in the terminal component of the path.

**FSPICK_NO_AUTOMOUNT**  
Don't follow automounts in the terminal component of the path.

**FSPICK_EMPTY_PATH**  
Allow an empty string to be specified as the pathname. This allows
*dirfd* to specify a path exactly.

After calling fspick(), the file descriptor should be passed to the
**fsconfig**(2) system call, using that to specify the desired changes
to filesystem and security parameters.

When the parameters are all set, the **fsconfig**() system call should
then be called again with **FSCONFIG_CMD_RECONFIGURE** as the command
argument to effect the reconfiguration.

After the reconfiguration has taken place, the context is wiped clean
(apart from the superblock attachment, which remains) and can be reused
to make another reconfiguration.

The file descriptor also serves as a channel by which more comprehensive
error, warning and information messages may be retrieved from the kernel
using **read**(2).

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

Messages are removed from the queue as they're read and the queue has a
limited depth, so it's possible for some to get lost.

# EXAMPLES

To illustrate the process, here's an example whereby this can be used to
reconfigure a filesystem:

    sfd = fspick(AT_FDCWD, "/mnt", FSPICK_NO_AUTOMOUNT | FSPICK_CLOEXEC);
    fsconfig(sfd, FSCONFIG_SET_FLAG, "ro", NULL, 0);
    fsconfig(sfd, FSCONFIG_SET_STRING, "user_xattr", "false", 0);
    fsconfig(sfd, FSCONFIG_CMD_RECONFIGURE, NULL, NULL, 0);

# RETURN VALUE

On success, the function returns a file descriptor. On error, -1 is
returned, and *errno* is set appropriately.

# ERRORS

The error values given below result from filesystem type independent
errors. Each filesystem type may have its own special errors and its own
special behavior. See the Linux kernel source code for details.

**EACCES**  
A component of a path was not searchable. (See also
**path_resolution**(7).)

**EFAULT**  
*pathname* points outside the user address space.

**EINVAL**  
*flags* includes an undefined value.

**ELOOP**  
Too many links encountered during pathname resolution.

**EMFILE**  
The system has too many open files to create more.

**ENFILE**  
The process has too many open files to create more.

**ENAMETOOLONG**  
A pathname was longer than **MAXPATHLEN**.

**ENOENT**  
A pathname was empty or had a nonexistent component.

**ENOMEM**  
The kernel could not allocate sufficient memory to complete the call.

**EPERM**  
The caller does not have the required privileges.

# CONFORMING TO

These functions are Linux-specific and should not be used in programs
intended to be portable.

# VERSIONS

**fsopen**(), **fsmount**() and **fspick**() were added to Linux in
kernel 5.1.

# NOTES

Glibc does not (yet) provide a wrapper for the **fspick**() system call;
call it using **syscall**(2).

# SEE ALSO

**mountpoint**(1), **fsconfig**(2), **fsopen**(2),
**path_resolution**(7), **mount**(8)
