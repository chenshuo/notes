# GCC

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

> [Quote:](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=54812#c10) The bug affects protected destructors too, and they're commonly used for base classes to prevent deletion via pointer-to-base.

Fixed in 4.9.0, by [r203985](https://gcc.gnu.org/viewcvs/gcc?view=revision&revision=203985).
Test case added by [r204057](https://gcc.gnu.org/viewcvs/gcc?view=revision&revision=204057).

# Glibc

### `strstr(3)`

Since glibc 2.9, strstr(3) may use [Two Way algorithm](http://www-igm.univ-mlv.fr/~lecroq/string/node26.html) in some cases, but the impl. had several bugs:

- [Bug 12092](https://sourceware.org/bugzilla/show_bug.cgi?id=12092)
- [Bug 12100](https://sourceware.org/bugzilla/show_bug.cgi?id=12100)
- [Bug 14602](https://sourceware.org/bugzilla/show_bug.cgi?id=14602)
- More?

# Kernel

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
