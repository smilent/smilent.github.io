---
layout: post
title: Files and Directories
tags:
- osx
- UNIX
comments: true
---

* toc
{:toc}
---

This blog is a note of APUE.
<br><br>

##File Types
The file types are:

1. Regular file. The most common type of file, which contains data of some form. There is no distinction to the UNIX kernel whether this data is text or binary. Any interpretation of the contents of a regular file is left to the application processing the file.

> One notable exception to this is with binary executable files. To execute a program, the kernel must understand its format. All binary executable files confirm to a format that allows the kernel to identify where to load a program's text and data.

2. Directory file. A file that contains the names of other files and pointers to information on these files. Any process that has read permission for a directory file can read the contents of the directory, but only the kernel can write directly to a directory file.

3. Block special file. A type of file providing buffered I/O access in fixed-size units to devices such as disk drivers.

4. Character special file. A type of file providing unbuffered I/O access in variable-sized units to devices. All devices on a system are either block special files or character special files.

5. FIFO. A type of file used for communication between processes. It's sometimes called a named pipe.

6. Socket. A type of file used for network communication between processes. A socket can also be used for non-network communication between processes on a single host.

7. Symbolic link. A type of file that points to another file.

The type of a file is encoded in the `st_mode` member of the <a href="#stat">`stat`</a> structure. We can determine the type of a file with the macros shown below. The argument to each macro is the `st_mode` member from the `stat` structure. 


|-----------------+------------|
| Macro           |Type of file|
|-----------------|------------|
| S_ISREG()       |regular file|
| S_ISDIR()       |directory file|
| S_ISCHR()       |character special file|
| S_ISBLK()       |block special file|
| S_ISFIFO()       |pipe or FIFO|
| S_ISLNK()       |symbolic link|
| S_ISSOCK()       |socket|
|-----------------+------------+

##User ID and Group ID

Every process has six or more IDs associated with it.

- The *real user ID* and *real group ID*: identify who we really are. They are taken from our entry in the password file when we log in.
- The *effective user ID*, *effective group ID* and *supplementary group IDs*: determine our file access permissions.
- The *saved set-user-ID* and *saved set-group-ID*: contain copies of the effective user ID and the effective group ID, respectively, when a program is executed.

Every file has an owner and a group owner, which are specified by `st_uid` and `st_gid` in <a href="#stat">`stat`</a>, respectively.

##File Access Permissions

The `st_mode` in <a href="#stat">`stat`</a> also encodes the access permission bits for the file. There are nine permission bits for each file, divided into three categories as shown below:

|-----------------+------------|
| <code>st_mode</code> mask  |Meaning     |
|-----------------|------------|
| S_IRUSR       |user-read|
| S_IWUSR       |user-write|
| S_IXUSR       |user-execute|
| S_IRGRP       |group-read|
| S_IWGRP       |group-write|
| S_IXGRP       |group-execute|
| S_IROTH       |other-read|
| S_IWOTH       |other-write|
| S_IXOTH       |other-execute|
|-----------------+------------+

Whenever we want to open any type of file by name, we must have execute permission in each ***directory*** (not the file itself) mentioned in the name, including the current directory, if it is implied. This is why the execute permission bit for a directory is often called the search bit.

We cannot create a new file in a directory unless we have write permission and execute permission in the directory.

To delete an existing file, we need write permission and execute permission in the directory containing the file. We *do not* need read permission or write permission for the file itself.

Execute permission for a file must be on if we want to execute the file using any of the seven `exec` functions. The file also has to be a regular file.

##Functions
Details of most functions can be obtained by <code>man 2 <i>func_name</i></code>.

###stat, fstate, fstatat, and lstat

<pre>
#include &lt;sys/stat.h&gt;
int stat(const char *restrict pathname, struct stat *restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *restrict pathname, struct stat *restrict buf);
int fstatat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag);
</pre>

All four return 0 if OK, -1 on error.

Given a *pathname*, the `stat` function returns a structure of information about the named file. The `lstat` function is similar to `stat`, but when the named file is a sumbolic link, `lstat` returns information about the symbolic link, not the file referenced by the symbolic link.

The *buf* argument is a pointer to a structure that we must supply, and the functions fill in the structure. The definition of the structure can differ among implementations. In OS X, it looks like:

<a id="stat"></a>
{% highlight c %}
struct stat { /* when _DARWIN_FEATURE_64_BIT_INODE is NOT defined */
    dev_t    st_dev;    /* device inode resides on */
    ino_t    st_ino;    /* inode's number */
    mode_t   st_mode;   /* inode protection mode */
    nlink_t  st_nlink;  /* number of hard links to the file */
    uid_t    st_uid;    /* user-id of owner */
    gid_t    st_gid;    /* group-id of owner */
    dev_t    st_rdev;   /* device type, for special file inode */
    struct timespec st_atimespec;  /* time of last access */
    struct timespec st_mtimespec;  /* time of last data modification */
    struct timespec st_ctimespec;  /* time of last file status change */
    off_t    st_size;   /* file size, in bytes */
    quad_t   st_blocks; /* blocks allocated for file */
    u_long   st_blksize;/* optimal file sys I/O ops blocksize */
    u_long   st_flags;  /* user defined flags for file */
    u_long   st_gen;    /* file generation number */
};

struct stat { /* when _DARWIN_FEATURE_64_BIT_INODE is defined */
    dev_t           st_dev;           /* ID of device containing file */
    mode_t          st_mode;          /* Mode of file (see below) */
    nlink_t         st_nlink;         /* Number of hard links */
    ino_t           st_ino;           /* File serial number */
    uid_t           st_uid;           /* User ID of the file */
    gid_t           st_gid;           /* Group ID of the file */
    dev_t           st_rdev;          /* Device ID */
    struct timespec st_atimespec;     /* time of last access */
    struct timespec st_mtimespec;     /* time of last data modification */
    struct timespec st_ctimespec;     /* time of last status change */
    struct timespec st_birthtimespec; /* time of file creation(birth) */
    off_t           st_size;          /* file size, in bytes */
    blkcnt_t        st_blocks;        /* blocks allocated for file */
    blksize_t       st_blksize;       /* optimal blocksize for I/O */
    uint32_t        st_flags;         /* user defined flags for file */
    uint32_t        st_gen;           /* file generation number */
    int32_t         st_lspare;        /* RESERVED: DO NOT USE! */
    int64_t         st_qspare[2];     /* RESERVED: DO NOT USE! */
};
{% endhighlight %}

###umask

<pre>
#include &lt;sys/stat.h&gt;
mode_t umask(mode_t cmask);
</pre>

Return previous file mode creation mask. It always succeeds.

Changing the file mode creation mask of a process doesn't affect the mask of its parent (often a shell). Users can set the `umask` value to control the default permissions on the files they create. This value is expressed in octal, with one bit representing one permission to be masked off, as shown below. Permissions can be denied by setting the corresponding bits. By default, OS X sets `umask` to 0022 so that files created by default don't have group-write or other-write permissions.

|---+---|
|Mask bit | Meaning|
|-------|----------|
|0400| user-read|
|0200| user-write|
|0100| user-execute|
|0040| group-read|
|0020| group-write|
|0010| group-execute|
|0004| other-read|
|0002| other-write|
|0001| other-execute|
|==========|=========|

###chmod, fchmod, and fchmodat

<pre>
#include &lt;sys/stat.h&gt;
int chmode(const char *pathname, mode_t mode);
int fchmod(int fd, mode_t mode);
int fchmodat(int fd, const char *pathname, mode_t mode, int flag);
</pre>

Return 0 if OK, -1 on error.

To change the permission bits of a file, the effective user ID of the process must be equal to the owner ID of the file, or the process must have superuser permissions.

###truncate

<pre>
#include &lt;unistd.h&gt;
int truncate(const char *pathname, off_t length);
int truncate(int fd, off_t length);
</pre>

Return 0 if OK, -1 on error.

These two functions truncate an existing file to *length* bytes. If the previous size of the file was greater than *length*, the data beyond *length* is no longer accessible. Otherwise, if the previous size was less than *length*, the file size will increase and the data between the old end of file and the new end of file will read as 0 (i.e., a hole is probably created in the file).

### link, linkat, unlink, unlinkat, and remove

<pre>
#include &lt;unistd.h&gt;
int link(const char *existingpath, const char *newpath);
int llinkat(int efd, const char *existingpath, int nfd, const char *newpath, int flag);
</pre>

Return 0 if OK, -1 on error.

A file can have multiple directory entries pointing to its i-node (<a href="#filesystem">Here</a> for more details). We can use either the `link` function or the `linkat` function to create a link to an existing file.

These functions create a new directory entry, *newpath*, that references the existing file *existingpath*. If the *newpath* already exists, an error is returned. Only the last component of the *newpath* is created. The rest of the path must already exist.

<pre>
#include &lt;unistd.h&gt;
int unlink(const char *pathname);
int unlinkat(int fd, const char *pathname, int flag);
</pre>

Return 0 if OK. -1 on error.

These functions remove the directory entry and decrement the link count of the file referenced by *pathname*. If there are other links to the file, the data in the file is still accessible though the other links. The file is not changed if an error occurs.

To unlink a file, **we must have write permission and execute permission in the directory containing the directory entry**, as it is the directory entry that we will be removing.

We can also unlink a file or a directory with the `remove` function. For a file, `remove` is identical to `unlink`. For a directory, `remove` is identical to `rmdir`.

<pre>
#include &lt;stdio.h&gt;
int remove(const char *pathname);
</pre>

Return 0 if OK, -1 on error.

###rename and renameat

<pre>
#include &lt;stdio.h&gt;
int rename(const char *oldname, const char *newname);
int renameat(int oldfd, const char *oldname, int newfd, const char *newname);
</pre>

Return 0 if OK, -1 on error.

A file or a directory is renamed with either the `rename` or `renameat` function.

There are several conditions to describe for these functions, depending on whether *oldname* refers to a file, a directory, or a symbolic link:

1. If *oldname* specifies a file that is not a directory, then we are renaming a file or a symbolic link. In this case, if *newname* exists, it cannot refer to a directory. If *newname* exists and is not a directory, it is removed, and *oldname* is renamed to *newname*.
2. If *oldname* specifies a directory, then we are renaming a directory. If *newname* exists, it must refer to a directory, and that directory **must be empty**. If *newname* exists and is an empty directory, it is removed, and *oldname* is renamed to *newname*. Additionally, when we're renaming a directory, *newname* cannot contain a path prefix that names *oldname*. For example, we can't rename */usr/foo* to *usr/foo/testdir*, because the old name (*/usr/foo*) is a path prefix of the new name and cannnot be removed.
3. If either *oldname* or *newname* refers to a symbolic link, then the link itself is processed, not the file to which it resolves.
4. We can't rename dot or dot-dot. More precisely, neither dot not dot-dot can appear as the last component of *oldname* or *newname*.
5. As a special case, if *oldname* and *newname* refer to the same file, the function returns successfully without changing anything.

If *newname* already exists, we need permissions as if we were deleting it. Also, because we're removing the directory entry for *oldname* and possibly creating a directory entry for *newname*, we need write permission and execute permission in the directory containing *oldname* and in the directory containing *newname*.

###symlink, symlinkat, readlink and readlinkat

<pre>
#include &lt;unistd.h&gt;
int symlink(const char *actualpath, const char *sympath);
int symlinkat(const char *actualpath, int fd, const char *sympath);
</pre>

Return 0 if OK, -1 on error.

A new directory entry, *sympath*, is created that points to *actualpath*. It is not required that *actualpath* exist when the symbolic link is created. Also, *actualpath* and *sympath* need not reside in the same file system.

<pre>
#include &lt;unistd.h&gt;
ssize_t readlink(const char *restrict pathname, char *restrict buf, size_t buf_size);
sszie_t readlinkat(int fd, const char *restrict pathname, char *restrict but, size_t bufsize);
</pre>

Unlike `open` function, which follows a symbolic link, these two functions open the link itself and read the name in the link.

###futimens, utimensat,amd utimes

<pre>
#include &ls;sys/stat.h&gt;
int futimens(int fd, const struct timespec times[2]);
int utimensat(int fd, const char *path, const struct timespec times[2], int flag);
</pre>

Return 0 if OK, -1 on error.

Both functions provide nanosecond granularity for specifying timestamps, using the `timespec` structure. The first element of the *times* array argument contains the access time, and the second element contains the modification time. The two time values are calendar times which count seconds since the Epoch.

Both functions are included in POSIX.1. A third function, `utimes`, is included in the Single UNIX Specification as part of the XSI option.

<pre>
#include &lt;sys/time.h&gt;
int utimes(const char *pathname, const struct timeval times[2]);
</pre>

Return 0 if OK, -1 on error.

{% highlight c %}
strcut timeval{
    time_t tv_sec; /* seconds */
    long   tv_usec; /* microseconds */
};
{% endhighlight %}

###mkdir, mkdirat and rmdir

<pre>
#include &lt;sys/stat.h&gt;
int mkdir(const char *pathname, mode_t mode);
int mkdirat(int fd, const char *pathname, mode_t mode);
</pre>

Return 0 if OK, -1 on error.

These functions create a new, empty directory. A common mistake is to specify the same *mode* as for a file: read and write permissions only. But for a directory, we normally want at least one of the execute bits enabled, to allow access to filenames within the directory.

<pre>
#include &lt;unistd.h&gt;
int rmdir(const char *pathname);
</pre>

Return 0 if OK, -1 on error. (Why are `mkdir` and `rmdir` defined in different headers?)

This function is to delete an *empty* directory. If any directory entries other than dot and dot-dot exist in the directory, the function fails. If the link count of the directory becomes 0 with this call, and if no other process has the directory open, then the space occupied by the directory is freed.

###reading directories

<pre>
#include &lt;dirent.h&gt;
DIR *opendir(const char *pathname);
DIR *fdopendir(int fd);
<div align="right">Both return: pointer if OK, NULL on error </div>
struct dirent *readdir(DIR *dp);
<div aligh="right">Returns: pointer if OK, NULL at end of directory or error</div>
void rewinddir(DIR *dp);
int closedir(DIR *dp);
<div align="right">Returns: 0 if OK, -1 on error</div>
ling telldir(DIR *dp);
<div align="right">Returns: current location in directory associated with <i>dp</i></div>
void seeldir(DIR *dp, long loc);
</pre>

Directories can be read by anyone who has access permission to read the directory. But only the kernel can write to a directory, to preserve file system sanity. The write permission bits and execute permission bits for a directory determine if we can create new files in the directory and remove files from the directory -- they don't specify if we can write to the directory itself.

The `dirent` structure defined in <dirent.h> is implementation dependent. Implementations define the structure to contain at least the following two members:
{% highlight c %}
ino_t d_ino; /* i-node number */
char d_name[]; /* null-terminated filename */
{% endhighlight %}

The `DIR` structure is an internal structure used by these seven functions to mainain information about the directory being read.

###chdir, fchdir, and getcwd

<pre>
#include &lt;unistd.h&gt;
int chdir(const char *pathname);
int fchdir(int fd);
</pre>

Return 0 if OK, -1 on error.

They are to change the current working directory of the calling process.

<pre>
#include &lt;unistd.h&gt;
char *getcwd(char *buf, size_t size);
</pre>

Return *buf* if OK, NULL on error.

It gets the current working directory of the calling process. We must pass to this function the address of a buffer, *buf*, and its *size* (in bytes). The buffer must be large enough to accommodate the absolute pathname plus a terminating null byte, or else an error will be returned.

##File Size

The `st_size` member of the <a href="#stat">`stat`</a> contains the size of the file in bytes. This field is meaningful only for regular files, directories, and symbolic links. For a regular file, a file size of 0 is allowed. We'll get an end-of-file indication on the first read of the file. For a directory, the file size is usually a multiple of a number, such as 16 or 512. For a symbolic link, the file size is the number of bytes in the file name.

Most contemporary UNIX systems provide the fields `st_blksize` and `st_blocks`. The first is the preferred block size for I/O for the file, and the latter is the actual number of 512-byte blocks that are allocated. The standard I/O library also tries to read or write `st_blksize` bytes at a time for efficiency.

<a id="filesystem"></a>

##File Systems

We can think of a disk drive being divided into one or more partitions. Each partition can contain a file system, as shown below. The i-nodes are *fixed-length* entries that contain most of the information about a file.
![Disk drive, partitions, and a file system](../images/disk_drive_partition_file_system.png)

If we examine the i-node and data block portion of a cylinder group in more detail, we could have the arrangement shown below.
![Cylinder group's i-nodes and data blocks in more detail](../images/i-node_data_block.png)

Note the following points from the figure above:

1. Two directory entries point to the same i-node entry. Every i-node has a link count that contains **the number of directory entries that point to it**. Only when the link count goes to 0 can the file be deleted (thereby releasing the data blocks associated with the file). In the <a href="#stat">`stat`</a> structure, the link count is contained in the `st_nlink` member.
2. The other type of link is called a *symbolic link*. With a symbolic link, the actual contexnts of the file -- the data blocks -- store the name of the file that the symbolic link points to. The file type in the i-node would be `S_IFLNK` so that the system knows that this is a symbolic link.
3. The i-node contains all the information about the file: the file type, the file's access permission bits, the size of the file, pointers to the file's data blocks, and so on. Most of the information in the `stat` structure is obtained from the i-node. Only two items of interest are stored in the directory entry: the filename and the i-node number. The data type for the i-node number is `ino_t`.
4. Because the i-node number in the directory entry points to an i-node in the same file system, **a directory entry can't refer to an i-node in a different file system.**
5. When renaming a file **without changing file systems**, the actual contents of the file need not be moved -- all that needs to be done is to add a new directory entry that points to the existing i-node and then unlink the old directory entry. The link count will remain the same. For example, to rename the file */usr/lib/foo* to */usr/foo*, the contents of the file *foo* need not be moved if the directories */usr/lib* and */usr* are on the same file system. This is how the `mv(1)` command usually operates.

What about the link count field for a directory? Assume that we make a new directory in the working directory, as in 
{% highlight bash %}
$ mkdir testdir
{% endhighlight %}

The figure below shows the result.
![Sample cylinder group after creating the directory testdir](../images/after_creating_testdir.png)

The i-node whose number id 2549 has a type field of "directory" and a link count equal to 2. Any leaf directory always has a link count of 2. The value of 2 comes from the directory entry that names the directory (*testdir* in this example) and from the entry for dot in that directory.  

##Symbolic Links

**A symbolic link is an indirect pointer to a file, unlike the hard links that point directly to the i-node of the file.** Symbolic links were introduced to get around he limitations of hard links.

- Hard links normally require that the link and the file reside in the same file system.
- Only the superuser can create a hard link to a directory (when supported by the underlying file system).

There are no file system limitations on a symbolic link and what it points to, and anyone can create a symbolic link to a directory. Symbolic links are typically used to "move" a file or an entire directory hierarchy to another location on a system.

A symbolic link does not affect the hard link count (`st_nlink`), while a hard link does. That is to say,
{% highlight bash %}
$ echo "this is for test" > a
$ ln -s a b # create a symbolic b that links to file a
$ rm a # assuming that there is no hard link to file a
$ cat b 
{% endhighlight %} 
will print 
{% highlight bash %}
$ cat: b : No such file or directory 
{% endhighlight %}

After `rm a`, the `st_nlink` of the correspond i-node is decreased to 0 (assuming there is no other hard link to file a), thus its data blocks will be released (see <a href="#filesystem">Here</a>). However,
{% highlight bash %}
$ echo "this is for test" > a
$ ln a b # create a hard link b that links to file a
$ rm a
$ cat b 
{% endhighlight %} 
will print 
{% highlight bash %}
$ this is for test
{% endhighlight %}

##File Times

|---|---|---|
|Field|Description|Example|ls(1) option|
|==|==|==|==|
|<code>st_atim</code>|last-access time of file data|<code>read</code>| <code>-u</code>|
|<code>st_mtim</code>|last-modification time of file data| <code>write</code>|default|
|<code>st_ctim</code>|last-change time of i-node status|<code>chmode</code>, <code>chown</code>|<code>-c</code>|
|======|======|========|

Note that the system does not maintain the last-access time for an i-node. This is why the functions `access` and `stat`, for example, don't change any of the three times.

##Summary of File Access Permission Bits

![summary of file access permission bits](../images/file_access_permission_bits.png)
