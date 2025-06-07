# qemu-arch

---

https://wiki.archlinux.org/title/Installation_guide

At the moment, the best option for archlinux on qemu is virtual machine images https://gitlab.archlinux.org/archlinux/arch-boxes/-/packages


```bash
image_dir=../images/archlinux
disk=${image_dir}/Arch-Linux-x86_64-basic-20250601.358142.qcow2
```

Using the virtual machine image is more convenient, because there is no lengthy installation process
```
$ qemu-system-x86_64 -smp 6 -m 2G -hda ${disk}
```
Run in another window to see the screen:



```bash
$ vncviewer :5900
```

Or try to add the user network

```bash
$ qemu-system-x86_64 -smp 6 -m 2G -hda ${disk} -net nic -net user,hostfwd=tcp::2222-:22
```
Then you can log into the system from the host:

```
$ ssh -p 2222 arch@localhost
You could also use sftp when needed:
```
You could automatically login when you copy your piblic key to the virtual machine
```bash
$ ssh-copy-id -p 2222 arch@localhost
```

```bash
$ sftp localhost -oPort=2222
```


