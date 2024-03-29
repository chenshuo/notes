# epoll

## Data structures

### eventpoll{}

### epitem{}
### eppoll_entry

## epoll_create()
```text
epoll_create1(flags)
  CHECK(only valid flags is O_CLOEXEC)
  struct eventpool* ep = ep_alloc();
  int fd = get_unused_fd_flags()
    -> __alloc_fd
      fd = find_next_fd()
      __set_open_fd(fd, fdt)
  struct file* file = anon_inode_getfile(name, fops, ep)
    path.dentry = d_alloc_pseudo()
    path.mnt = mntget()
    d_instantiate(path.dentry)
    file = alloc_file(&path, flags, fops)
    file->private_data = ep

  ep->file = file
  fd_install(fd, file)
  return fd
```

## close()

```text
close(fd)
  -> __close_fd(current->files, fd)
    struct fdtable* fdt = files_fdtable(files)
    struct file* file = fdt->fd[fd]
    fdt->fd[fd] = NULL
    -> __put_unused_fd(files, fd)
      -> __clear_open_fd(fd, fdt)  // clear bit masks
    -> filp_close(file, files)
      -> fput(file)
        if (file_count(file) == 0) return 0
        if (--file->f_count == 0)
          __fput(file)  // delayed
            -> file->f_op->release(inode, file)
              struct eventpoll *ep = file->private_data
              if (ep) ep_free(ep)
                -> ep_remove(ep, epi)
                  -> epi_rcu_free

```

## epoll_ctl(ADD/DEL/MOD)


```text
epoll_ctl(epfd, op, fd event)
  f = fdget(epfd)  // epoll itself
  tf = fdget(fd)   // target
  eventpoll* ep = f.file->private_data
  if (ADD)
    -> ep_look_check(ep, tf.file)
  epitem* epi = ep_find(ep, tf.file, fd)
  if (ADD && epi == NULL)
    -> ep_insert(ep, event, tf.file, fd, )
  if (DEL && epi != NULL)
    -> ep_remove(ep, epi)
  if (MOD && epi != NULL && !(event & EXCLUSIVE))
    -> ep_modify(ep, epi, event)
```

## `ep_insert`

```text
  -> ep_insert
```


## `ep_remove`

```text
  -> ep_remove
```

## `ep_modify`

```text
  -> ep_modify
```

## epoll_wait()
