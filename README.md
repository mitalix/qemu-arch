# qemu-arch

---

Create a disk drive, that is a file image:

https://wiki.archlinux.org/title/Installation_guide

```bash
image_dir=../images/archlinux
disk=${image_dir}/Arch-Linux-x86_64-basic-20250601.358142.qcow2
```

Run qemu and wait for installer to come up:
```
$ qemu-system-x86_64 -smp 6 -m 2G -hda ${disk}
```
Run in another window to see the screen:



```bash
$ vncviewer :5900
```

Not all options are needed especially when creating a test virtual machine, but one of them is a nice convenience
```
# setfont ter-132b
```





Or proceed to the install, but try to add the user network

```bash
$ qemu-system-x86_64 -smp 6 -m 2G -hda ${disk} -net nic -net user,hostfwd=tcp::2222-:22

```
Then you can log into the system from the host:

```
# host > ssh -p 2222 italix@localhost
You could also use sftp when needed:
```

```bash
$ sftp localhost -oPort=2222
```


