# qemu-arch

---

Create a disk drive, that is a file image:

```bash
$ qemu-img create mydisk.img 10G
```
Run qemu and wait for installer to come up:
```
$ qemu-system-x86_64 -smp 6 -m 2G -hda ../images/archlinux/mydisk.img -cdrom ../images/archlinux/archlinux-2025.06.01-x86_64.iso

```
Run in another window to see the screen:



```bash
$ vncviewer :5900
```

Not all options are needed especially when creating a test virtual machine, but one of them is a nice convenience
```bash
# setfont ter-132b
```




```php
$ qemu-system-x86_64 -hda test.img -smp 4 -m 4G -accel kvm 
```


