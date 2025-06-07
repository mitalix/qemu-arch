# qemu-arch

---

- Install qemu on host machine
- Install or acquire VM image for guest machine
- Kubernetes
    - docker
    - minikube (requires at least 2G memory)



https://wiki.archlinux.org/title/Installation_guide

Presently, the best option for archlinux on qemu is a virtual machine image https://gitlab.archlinux.org/archlinux/arch-boxes/-/packages


```bash
image_dir=../images/archlinux
disk=${image_dir}/Arch-Linux-x86_64-basic-20250601.358142.qcow2
```

Using the virtual machine image is more convenient, because there is no lengthy installation process
```bash
$ qemu-system-x86_64 -smp 6 -m 2G -hda ${disk}
```
Run in another window to see the screen:
```bash
$ vncviewer :5900
```
Make sure openssh is installed, if you want to ssh to the user network
```bash
$ qemu-system-x86_64 -smp 6 -m 4G -hda ${disk} -net nic -net user,hostfwd=tcp::2222-:22
```
The username/password default  combination for the base VM is arch/arch. Log into the system from the host:

```bash
$ ssh -p 2222 arch@localhost
```
You could automatically login when you copy your piblic key from the host to the guest virtual machine
```bash
$ ssh-copy-id -p 2222 arch@localhost
```
Sftp also works

```bash
$ sftp arch@localhost -oPort=2222
```

Network seems fine. So, do an upgrade

```bash
[arch@archlinux ~]$ sudo su pacman -Syu
```


