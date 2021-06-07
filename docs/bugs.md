# Interesting bugs in open source projects

## GCC

### [Bug 54812](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=54812): `private` `=default` destructor is `public`

Delete expression doesn't respect access of defaulted destructor

```cpp
struct Base
{
 private:
  ~Base() = default;
};

int main()
{
  Base* p = new Base;
  delete p;
}
```

> [Quote:](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=54812#c10) _The bug affects protected destructors too, and they're commonly used for base classes to prevent deletion via pointer-to-base._

Fixed in 4.9.0, by [r203985](https://gcc.gnu.org/viewcvs/gcc?view=revision&revision=203985).
Test case added by [r204057](https://gcc.gnu.org/viewcvs/gcc?view=revision&revision=204057).

## Glibc

### `strstr(3)`

Since glibc 2.9, strstr(3) may use [Two Way algorithm](http://www-igm.univ-mlv.fr/~lecroq/string/node26.html) in some cases, but the impl. had several bugs:

- [Bug 12092](https://sourceware.org/bugzilla/show_bug.cgi?id=12092)
- [Bug 12100](https://sourceware.org/bugzilla/show_bug.cgi?id=12100)
- [Bug 14602](https://sourceware.org/bugzilla/show_bug.cgi?id=14602)
- More?

### `getpid(2)` no longer caches PID

Since [glibc 2.25](https://sourceware.org/glibc/wiki/Release/2.25), `getpid(2)` no longer caches PID,
released on 2017-02-05.  Ubuntu 18.04, Debian 10 and CentOS 8 contain this change.

> Quote from [`man 2 getpid`](https://man7.org/linux/man-pages/man2/getpid.2.html):
    _From glibc version 2.3.4 up to and including version 2.24, the glibc wrapper function for getpid() cached PIDs,
    with the goal of avoiding additional system calls when a process calls getpid() repeatedly.
    ..._

> _Because of the aforementioned problems, since glibc version 2.25, the PID cache is removed:
    calls to getpid() always invoke the actual system call, rather than returning a cached value._

In a tight loop, it takes about 200ns for `getpid(2)` on z420 (E5-1620 3.6GHz) running Linux 5.10.

## Kernel

### `write(2)` not thread-safe

Fixed in v3.14, released in 2014-03-30. Ubuntu 14.04 has this bug, Debian 8 doesn't.

```text
commit 9c225f2655e36a470c4f58dbbc99244c5fc7f2d4
Author: Linus Torvalds <torvalds@linux-foundation.org>
Date:   Mon Mar 3 09:36:58 2014 -0800

    vfs: atomic f_pos accesses as per POSIX

    Our write() system call has always been atomic in the sense that you get
    the expected thread-safe contiguous write, but we haven't actually
    guaranteed that concurrent writes are serialized wrt f_pos accesses, so
    threads (or processes) that share a file descriptor and use "write()"
    concurrently would quite likely overwrite each others data.

    This violates POSIX.1-2008/SUSv4 Section XSI 2.9.7 that says:

     "2.9.7 Thread Interactions with Regular File Operations

      All of the following functions shall be atomic with respect to each
      other in the effects specified in POSIX.1-2008 when they operate on
      regular files or symbolic links: [...]"

    and one of the effects is the file position update.

    This unprotected file position behavior is not new behavior, and nobody
    has ever cared.  Until now.  Yongzhi Pan reported unexpected behavior to
    Michael Kerrisk that was due to this.

    This resolves the issue with a f_pos-specific lock that is taken by
    read/write/lseek on file descriptors that may be shared across threads
    or processes.

    Reported-by: Yongzhi Pan <panyongzhi@gmail.com>
    Reported-by: Michael Kerrisk <mtk.manpages@gmail.com>
    Cc: Al Viro <viro@zeniv.linux.org.uk>
    Signed-off-by: Linus Torvalds <torvalds@linux-foundation.org>
    Signed-off-by: Al Viro <viro@zeniv.linux.org.uk>
```


`$ man 2 write`
```text
BUGS
       According  to  POSIX.1-2008/SUSv4  Section XSI 2.9.7 ("Thread Interactions
       with Regular File Operations"):

           All of the following functions shall be atomic with  respect  to  each
           other  in  the  effects specified in POSIX.1-2008 when they operate on
           regular files or symbolic links: ...

       Among the APIs subsequently listed are write() and writev(2).   And  among
       the  effects  that  should  be  atomic  across threads (and processes) are
       updates of the file offset.  However, on Linux before version  3.14,  this
       was  not  the  case:  if two processes that share an open file description
       (see open(2)) perform a write() (or writev(2)) at the same time, then  the
       I/O operations were not atomic with respect updating the file offset, with
       the result that the blocks of data  output  by  the  two  processes  might
       (incorrectly) overlap.  This problem was fixed in Linux 3.14.
```

## Java

### ["+=" applied to String operands can provoke side effects](https://bugs.openjdk.java.net/browse/JDK-8204322)

Introduced in JDK9, fixed in JDK10 but not JDK9.

[Why does array[idx++]+=“a” increase idx once in Java 8 but twice in Java 9 and 10?](https://stackoverflow.com/questions/50683786/why-does-arrayidx-a-increase-idx-once-in-java-8-but-twice-in-java-9-and-1)
