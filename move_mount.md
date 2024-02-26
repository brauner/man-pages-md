# NAME

move_mount - Move mount objects around the filesystem topology

# SYNOPSIS

    #include <sys/types.h>
    #include <sys/mount.h>
    #include <unistd.h>
    #include <fcntl.h> /* Definition of AT_* constants */

    int move_mount(int from_dirfd, const char *from_pathname,
     int to_dirfd, const char *to_pathname,
     unsigned int flags);

*Note*: There is no glibc wrapper for this system call.

# DESCRIPTION

The **move_mount**() call moves a mount from one place to another; it
can also be used to attach an unattached mount created by **fsmount**()
or **open_tree**() with **OPEN_TREE_CLONE**.

If **move_mount**() is called repeatedly with a file descriptor that
refers to a mount object, then the object will be attached/moved the
first time and then moved again and again and again, detaching it from
the previous mountpoint each time.

To access the source mount object or the destination mountpoint, no
permissions are required on the object itself, but if either pathname is
supplied, execute (search) permission is required on all of the
directories specified in *from_pathname* or *to_pathname*.

The caller does, however, require the appropriate capabilities or
permission to effect a mount.

**move_mount**() uses *from_pathname*, *from_dirfd* and part of *flags*
to locate the mount object to be moved and *to_pathname*, *to_dirfd* and
another part of *flags* to locate the destination mountpoint. Each
lookup can be done in one of a variety of ways:

\[\*\] By absolute path.  
The pathname points to an absolute path and the dirfd is ignored. The
file is looked up by name, starting from the root of the filesystem as
seen by the calling process.

\[\*\] By cwd-relative path.  
The pathname points to a relative path and the dirfd is *AT_FDCWD*. The
file is looked up by name, starting from the current working directory.

\[\*\] By dir-relative path.  
The pathname points to relative path and the dirfd indicates a file
descriptor pointing to a directory. The file is looked up by name,
starting from the directory specified by *dirfd*.

\[\*\] By file descriptor.  
The pathname points to "", the dirfd points directly to the mount object
to move or the destination mount point and the appropriate
**\*\_EMPTY_PATH** flag is set.

*flags* can be used to influence a path-based lookup. A value for
*flags* is constructed by OR'ing together zero or more of the following
constants:

**MOVE_MOUNT_F_EMPTY_PATH**  
If *from_pathname* is an empty string, operate on the file referred to
by *from_dirfd* (which may have been obtained using the **open**(2)
**O_PATH** flag or **open_tree**()) If *from_dirfd* is **AT_FDCWD**, the
call operates on the current working directory. In this case,
*from_dirfd* can refer to any type of file, not just a directory. This
flag is Linux-specific; define **\_GNU_SOURCE** to obtain its
definition.

**MOVE_MOUNT_T_EMPTY_PATH**  
As above, but operating on *to_pathname* and *to_dirfd*.

**MOVE_MOUNT_F_NO_AUTOMOUNT**  
Don't automount the terminal ("basename") component of *from_pathname*
if it is a directory that is an automount point. This allows a mount
object that has an automount point at its root to be moved and prevents
unintended triggering of an automount point. The
**MOVE_MOUNT_F_NO_AUTOMOUNT** flag has no effect if the automount point
has already been mounted over. This flag is Linux-specific; define
**\_GNU_SOURCE** to obtain its definition.

**MOVE_MOUNT_T_NO_AUTOMOUNT**  
As above, but operating on *to_pathname* and *to_dirfd*. This allows an
automount point to be manually mounted over.

**MOVE_MOUNT_F_SYMLINKS**  
If *from_pathname* is a symbolic link, then dereference it. The default
for **move_mount**() is to not follow symlinks.

**MOVE_MOUNT_T_SYMLINKS**  
As above, but operating on *to_pathname* and *to_dirfd*.

**MOVE_MOUNT_SET_GROUP** (since Linux 5.15)  
Add an existing private mount into a propagation group. This makes it
possible to first create a mount tree consisting only of private mounts
and configuring the desired propagation layout afterwards.

# EXAMPLES

The **move_mount**() function can be used like the following:

>     move_mount(AT_FDCWD, "/a", AT_FDCWD, "/b", 0);

This would move the object mounted on "/a" to "/b". It can also be used
in conjunction with **open_tree**(2) or **open**(2) with **O_PATH**:

>     fd = open_tree(AT_FDCWD, "/mnt", 0);
>     move_mount(fd, "", AT_FDCWD, "/mnt2", MOVE_MOUNT_F_EMPTY_PATH);
>     move_mount(fd, "", AT_FDCWD, "/mnt3", MOVE_MOUNT_F_EMPTY_PATH);
>     move_mount(fd, "", AT_FDCWD, "/mnt4", MOVE_MOUNT_F_EMPTY_PATH);

This would attach the path point for "/mnt" to fd, then it would move
the mount to "/mnt2", then move it to "/mnt3" and finally to "/mnt4".

It can also be used to attach new mounts:

>     sfd = fsopen("ext4", FSOPEN_CLOEXEC);
>     fsconfig(sfd, FSCONFIG_SET_STRING, "source", "/dev/sda1", 0);
>     fsconfig(sfd, FSCONFIG_SET_FLAG, "user_xattr", NULL, 0);
>     fsconfig(sfd, FSCONFIG_CMD_CREATE, NULL, NULL, 0);
>     mfd = fsmount(sfd, FSMOUNT_CLOEXEC, MOUNT_ATTR_NODEV);
>     move_mount(mfd, "", AT_FDCWD, "/home", MOVE_MOUNT_F_EMPTY_PATH);

Which would open the Ext4 filesystem mounted on "/dev/sda1", turn on
user extended attribute support and create a mount object for it.
Finally, the new mount object would be attached with **move_mount**() to
"/home".

# RETURN VALUE

On success, 0 is returned. On error, -1 is returned, and *errno* is set
appropriately.

# ERRORS

**EACCES**  
Search permission is denied for one of the directories in the path
prefix of *pathname*. (See also **path_resolution**(7).)

**EBADF**  
*from_dirfd* or *to_dirfd* is not a valid open file descriptor.

**EFAULT**  
*from_pathname* or *to_pathname* is NULL or either one point to a
location outside the process's accessible address space.

**EINVAL**  
Reserved flag specified in *flags*.

**ELOOP**  
Too many symbolic links encountered while traversing the pathname.

**ENAMETOOLONG**  
*from_pathname* or *to_pathname* is too long.

**ENOENT**  
A component of *from_pathname* or *to_pathname* does not exist, or one
is an empty string and the appropriate **\*\_EMPTY_PATH** was not
specified in *flags*.

**ENOMEM**  
Out of memory (i.e., kernel memory).

**ENOTDIR**  
A component of the path prefix of *from_pathname* or *to_pathname* is
not a directory or one or the other is relative and the appropriate
*\*\_dirfd* is a file descriptor referring to a file other than a
directory.

# VERSIONS

**move_mount**() was added to Linux in kernel 4.18.

# CONFORMING TO

**move_mount**() is Linux-specific.

# NOTES

Glibc does not (yet) provide a wrapper for the **move_mount**() system
call; call it using **syscall**(2).

# SEE ALSO

**fsmount**(2), **fsopen**(2), **open_tree**(2)
