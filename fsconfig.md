# NAME

fsconfig - Filesystem parameterisation

# SYNOPSIS

    #include <sys/types.h>
    #include <sys/mount.h>
    #include <unistd.h>

    int fsconfig(int *fd, unsigned int cmd, const char *key,
     const void __user *value, int aux);

*Note*: There is no glibc wrapper for this system call.

# DESCRIPTION

**fsconfig**() is used to supply parameters to and issue commands
against a filesystem configuration context as set up by **fsopen**(2) or
**fspick**(2). The context is supplied attached to the file descriptor
specified by *fd* argument.

The *cmd* argument indicates the command to be issued, where some of the
commands simply supply parameters to the context. The meaning of *key*,
*value* and *aux* are command-dependent; unless required for the
command, these should be set to NULL or 0.

The available commands are:

**FSCONFIG_SET_FLAG**  
Set the parameter named by *key* to true. This may incur error
**EINVAL** if the parameter requires an argument.

**FSCONFIG_SET_STRING**  
Set the parameter named by *key* to a string. This may incur error
**EINVAL** if the parser doesn't want a parameter here, wants a
non-string or the string cannot be interpreted appropriately. *value*
points to a NUL-terminated string.

**FSCONFIG_SET_BINARY**  
Set the parameter named by *key* to be a binary blob argument. This may
cause **EINVAL** to be returned if the filesystem parser isn't expecting
a binary blob and it can't be converted to something usable. *value*
points to the data and *aux* indicates the size of the data.

**FSCONFIG_SET_PATH**  
Set the parameter named by *key* to the object at the provided path.
*value* should point to a NULL-terminated pathname string and aux may
indicate **AT_FDCWD** or a file descriptor indicating a directory from
which to begin a relative pathwalk. This may return any errors incurred
by the pathwalk and may return **EINVAL** if the parameter isn't
expecting a path.

Note that FSCONFIG_SET_STRING can be used instead, implying AT_FDCWD.

**FSCONFIG_SET_PATH_EMPTY**  
As FSCONFIG_SET_PATH, but with **AT_EMPTY_PATH** applied to the
pathwalk.

**FSCONFIG_SET_FD**  
Set the parameter named by *key* to the file descriptor specified by
*aux*. This will incur **EINVAL** if the parameter doesn't expect a file
descriptor or **EBADF** if the file descriptor is invalid.

Note that FSCONFIG_SET_STRING can be used instead with the file
descriptor passed as a decimal string.

**FSCONFIG_CMD_CREATE**  
This command causes the filesystem to take the parameters set in the
context and to try to create filesystem representation in the kernel. If
it can share an existing one, it may do that instead if the filesystem
type and parameters permit that. This is intended for use with
**fsopen**(2).

**FSCONFIG_CMD_CREATE_EXCL** (since Linux 6.6)  
This command is imilar to **FSCONFIG_CMD_CREATE** but will return
**EBUSY** if a matching superblock already exists. Userspace that needs
to be sure that it did create a new superblock with the requested mount
options can request superblock creation using this command. If the
command succeeds they can be sure that they did create a new superblock
with the requested mount options.

**FSCONFIG_CMD_RECONFIGURE**  
This command causes the filesystem to apply the parameters set in the
context to an already existing filesystem representation in memory and
to alter it. This is intended for use with **fspick**(2), but may also
by used against the context created by **fsopen()** after **fsmount**(2)
has been called on it.

# EXAMPLES

    fsconfig(sfd, FSCONFIG_SET_FLAG, "ro", NULL, 0);

    fsconfig(sfd, FSCONFIG_SET_STRING, "user_xattr", "false", 0);

    fsconfig(sfd, FSCONFIG_SET_BINARY, "ms_pac", pac_buffer, pac_size);

    fsconfig(sfd, FSCONFIG_SET_PATH, "journal", "/dev/sdd4", AT_FDCWD);

    dirfd = open("/dev/", O_PATH);
    fsconfig(sfd, FSCONFIG_SET_PATH, "journal", "sdd4", dirfd);

    fd = open("/overlays/mine/", O_PATH);
    fsconfig(sfd, FSCONFIG_SET_PATH_EMPTY, "lower_dir", "", fd);

    pipe(pipefds);
    fsconfig(sfd, FSCONFIG_SET_FD, "fd", NULL, pipefds[1]);

# RETURN VALUE

On success, the function returns 0. On error, -1 is returned, and
*errno* is set appropriately.

# ERRORS

The error values given below result from filesystem type independent
errors. Each filesystem type may have its own special errors and its own
special behavior. See the Linux kernel source code for details.

**EACCES**  
A component of a path was not searchable. (See also
**path_resolution**(7).)

**EACCES**  
Mounting a read-only filesystem was attempted without specifying the
'**ro**' parameter.

**EACCES**  
A specified block device is located on a filesystem mounted with the
**MS_NODEV** option.

**EBADF**  
The file descriptor given by *fd* or possibly by *aux* (depending on the
command) is invalid.

**EBUSY**  
The context attached to *fd* is in the wrong state for the given
command.

**EBUSY**  
The filesystem representation cannot be reconfigured read-only because
it still holds files open for writing.

**EBUSY**  
A new superblock was requested with **FSCONFIG_CMD_CREATE_EXCL** but a
matching superblock already existed.

**EFAULT**  
One of the pointer arguments points outside the user address space.

**EINVAL**  
*fd* does not refer to a filesystem configuration context.

**EINVAL**  
One of the source parameters referred to an invalid superblock.

**ELOOP**  
Too many links encountered during pathname resolution.

**ENAMETOOLONG**  
A path name was longer than **MAXPATHLEN**.

**ENOENT**  
A pathname was empty or had a nonexistent component.

**ENOMEM**  
The kernel could not allocate sufficient memory to complete the call.

**ENOTBLK**  
Once of the parameters does not refer to a block device (and a device
was required).

**ENOTDIR**  
*pathname*, or a prefix of *source*, is not a directory.

**EOPNOTSUPP**  
The command given by *cmd* was not valid.

**ENXIO**  
The major number of a block device parameter is out of range.

**EPERM**  
The caller does not have the required privileges.

# CONFORMING TO

These functions are Linux-specific and should not be used in programs
intended to be portable.

# VERSIONS

**fsconfig**() was added to Linux in kernel 5.1.

# NOTES

Glibc does not (yet) provide a wrapper for the **fspick**() system call;
call it using **syscall**(2).

# SEE ALSO

**mountpoint**(1), **fsmount**(2), **fsopen**(2), **fspick**(2),
**mount_namespaces**(7), **path_resolution**(7)
