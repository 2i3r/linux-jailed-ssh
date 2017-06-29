# How to create jailed ssh session:

__1- create group and user:__

```
groupadd sshjailedusers
useradd -g sshjailedusers newuser
```

__2- create jailed root structure__

```
root=/home/jailedroot
mkdir ${root}
mkdir -p ${root}/{dev,etc,lib,usr,bin,proc,sys,home}
mkdir ${root}/usr/bin
mkdir ${root}/dev/pts
chown root:root ${root}
```

__3- mount proc. sys, dev and optionally run__

```
cd ${root}/
mount -t proc proc proc/
mount -o bind /sys sys/
mount -o bind /dev dev/
mount -o bind /dev/pts dev/pts/
mount -o bind /run run/
```

Instead of full dev bind making only "null,  zero, stdin, stdout, stderr, random, tty"
may be sufficient, by something like this:

```
mknod -m 666 ${root}/dev/null c 1 3
mknod -m 666 ${root}/dev/tty c 5 0
mknod -m 666 ${root}/dev/zero c 1 5
mknod -m 666 ${root}/dev/random c 1 8
...
..
.
```

__4- fill up etc directory with minimum files:__

```
cd ${root}/etc
cp /etc/ld.so.cache .
cp /etc/ld.so.conf .
cp /etc/nsswitch.conf .
cp /etc/hosts .
```
> (optional ↓↓)
```
cp /etc/profile{,.d} .
cp /etc/passwd .
cp /etc/group .
```

__5- copy some binaries for command you want to ${root}/usr/bin and also to ${root}/bin__
bash, cp, head, less, ls, mv, tail, tty, whoami are some examples

__6- using l2chroot, you can copy needed libraries for each binary__
replace "BASE" variable initialization in "l2chroot" bash file with your jailed root path

```
cd /sbin
```

then for each binary you copied, do:

```
l2chroot [command_name]
```

__7- l2chroot may not copy all the needed files, you need to "strace" each binary __
and look at "open" addresses, and copy those file it they has not:
for example for bash:

```
strace bash -c ";" 2>&1 | grep -e "^open"
```

__8- now everything should be there, configure /etc/ssh/sshd_config__
add these lines to /etc/ssh/sshd_config

```
Match Group sshusers
    AllowUsers newuser
    ChrootDirectory /home/jailedroot
    #ForceCommand internal-sftp
    X11Forwarding no
    AllowTcpForwarding no
```

__9- restart sshd service__

```
systemctl restart sshd
```

__10- Optionally create home directories for newuser__

```
cd ${root}/home
mkdir newuser
cp /etc/skel newuser/.bashrc
chmod -R 755 newuser/
chown -R newuser:sshjaileduser newuser/
```


##### NOTE:
This might be much more than needed (full sys, dev) for limited applications
perhaps with restricted user, it may not be a serious problem. should look more 
into it.
**any commit and issue is appreciated.**
