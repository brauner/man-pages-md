# NAME

mount_attr - what mount properties to set and clear

# LIBRARY

Linux kernel headers

# SYNOPSIS

    #include <sys/mount.h>
    struct mount_attr {
     u64 attr_set; /* Mount properties to set */
     u64 attr_clr; /* Mount properties to clear */
     u64 propagation; /* Mount propagation type */
     u64 userns_fd; /* User namespace file descriptor */
        /* ... */
    };

# DESCRIPTION

Specifies which mount properties should be changed with
**mount_setattr**(2).

The fields are as follows:

*.attr_set*  
This field specifies which **MOUNT_ATTR\_***\** attribute flags to set.

*.attr_clr*  
This field specifies which **MOUNT_ATTR\_***\** attribute flags to
clear.

*.propagation*  
This field specifies what mount propagation will be applied. The valid
values of this field are the same propagation types described in
**mount_namespaces**(7).

*.userns_fd*  
This field specifies a file descriptor that indicates which user
namespace to use as a reference for ID-mapped mounts with
**MOUNT_ATTR_IDMAP**.

# STANDARDS

Linux.

# HISTORY

Linux 5.12. glibc 2.36.

Extra fields may be appended to the structure, with a zero value in a
new field resulting in the kernel behaving as though that extension
field was not present. Therefore, a user *must* zero-fill this structure
on initialization.

# SEE ALSO

**mount_setattr**(2)
