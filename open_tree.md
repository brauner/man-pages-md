# NAME

open_tree - Pick or clone mount object and attach to fd

# SYNOPSIS

    #include <sys/types.h>


    #include <sys/mount.h>


    #include <unistd.h>


    #include <fcntl.h> /* Definition of AT_* constants */

    int open_tree(int dirfd, const char *pathname, unsigned int flags);

*Note*: There are no glibc wrappers for these system calls.

# DESCRIPTION

**open_tree**() picks the mount object specified by the pathname and
attaches it to a new file descriptor or clones it and attaches the clone
to the file descriptor. The resultant file descriptor is
indistinguishable from one produced by **open**(2) with **O_PATH**.

In the case that the mount object is cloned, the clone will be
"unmounted" and destroyed when the file descriptor is closed if it is
not otherwise mounted somewhere by calling **move_mount**(2).

To select a mount object, no permissions are required on the object
referred to by the path, but execute (search) permission is required on
all of the directories in *pathname* that lead to the object.

To clone an object, however, the caller must have mount capabilities and
permissions.

**open_tree**() uses *pathname*, *dirfd* and *flags* to locate the
target object in one of a variety of ways:

\[\*\] By absolute path.  
*pathname* points to an absolute path and *dirfd* is ignored. The object
is looked up by name, starting from the root of the filesystem as seen
by the calling process.

\[\*\] By cwd-relative path.  
*pathname* points to a relative path and *dirfd* is *AT_FDCWD*. The
object is looked up by name, starting from the current working
directory.

\[\*\] By dir-relative path.  
*pathname* points to relative path and *dirfd* indicates a file
descriptor pointing to a directory. The object is looked up by name,
starting from the directory specified by *dirfd*.

\[\*\] By file descriptor.  
*pathname* is "", *dirfd* indicates a file descriptor and
**AT_EMPTY_PATH** is set in *flags*. The mount attached to the file
descriptor is queried directly. The file descriptor may point to any
type of file, not just a directory.

*flags* can be used to control the operation of the function and to
influence a path-based lookup. A value for *flags* is constructed by
OR'ing together zero or more of the following constants:

**AT_EMPTY_PATH**  
If *pathname* is an empty string, operate on the file referred to by
*dirfd* (which may have been obtained from **open**(2) with **O_PATH**,
from **fsmount**(2) or from another **open_tree**()). If *dirfd* is
**AT_FDCWD**, the call operates on the current working directory. In
this case, *dirfd* can refer to any type of file, not just a directory.
This flag is Linux-specific; define **\_GNU_SOURCE** to obtain its
definition.

**AT_NO_AUTOMOUNT**  
Don't automount the terminal ("basename") component of *pathname* if it
is a directory that is an automount point. This flag allows the
automount point itself to be picked up or a mount cloned that is rooted
on the automount point. The **AT_NO_AUTOMOUNT** flag has no effect if
the mount point has already been mounted over. This flag is
Linux-specific; define **\_GNU_SOURCE** to obtain its definition.

**AT_SYMLINK_NOFOLLOW**  
If *pathname* is a symbolic link, do not dereference it: instead pick up
or clone a mount rooted on the link itself.

**OPEN_TREE_CLOEXEC**  
Set the close-on-exec flag for the new file descriptor. This will cause
the file descriptor to be closed automatically when a process exec's.

**OPEN_TREE_CLONE**  
Rather than directly attaching the selected object to the file
descriptor, clone the object, set the root of the new mount object to
that point and attach the clone to the file descriptor.

**AT_RECURSIVE**  
This is only permitted in conjunction with OPEN_TREE_CLONE. It causes
the entire mount subtree rooted at the selected spot to be cloned rather
than just that one mount object.

# EXAMPLE

The **open_tree**() function can be used like the following:

>     fd1 = open_tree(AT_FDCWD, "/mnt", 0);
>     fd2 = open_tree(fd1, "",
>                     AT_EMPTY_PATH | OPEN_TREE_CLONE | AT_RECURSIVE);
>     move_mount(fd2, "", AT_FDCWD, "/mnt2", MOVE_MOUNT_F_EMPTY_PATH);

This would attach the path point for "/mnt" to fd1, then it would copy
the entire subtree at the point referred to by fd1 and attach that to
fd2; lastly, it would attach the clone to "/mnt2".

# RETURN VALUE

On success, the new file descriptor is returned. On error, -1 is
returned, and *errno* is set appropriately.

# ERRORS

**EACCES**  
Search permission is denied for one of the directories in the path
prefix of *pathname*. (See also **path_resolution**(7).)

**EBADF**  
*dirfd* is not a valid open file descriptor.

**EFAULT**  
*pathname* is NULL or *pathname* point to a location outside the
process's accessible address space.

**EINVAL**  
Reserved flag specified in *flags*.

**ELOOP**  
Too many symbolic links encountered while traversing the pathname.

**ENAMETOOLONG**  
*pathname* is too long.

**ENOENT**  
A component of *pathname* does not exist, or *pathname* is an empty
string and **AT_EMPTY_PATH** was not specified in *flags*.

**ENOMEM**  
Out of memory (i.e., kernel memory).

**ENOTDIR**  
A component of the path prefix of *pathname* is not a directory or
*pathname* is relative and *dirfd* is a file descriptor referring to a
file other than a directory.

# VERSIONS

**open_tree**() was added to Linux in kernel 4.18.

# CONFORMING TO

**open_tree**() is Linux-specific.

# NOTES

Glibc does not (yet) provide a wrapper for the **open_tree**() system
call; call it using **syscall**(2).

# SEE ALSO

**fsmount**(2), **move_mount**(2), **open**(2)
