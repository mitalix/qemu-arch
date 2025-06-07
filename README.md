# qemu-arch

---

- Install qemu on host machine
- Install or acquire VM image for guest machine
- Kubernetes
    - docker
    - minikube (requires at least 2G memory)
    - kubectl



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
You could automatically login when you copy your public key from the host to the guest virtual machine
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


Install docker

```bash
[arch@archlinux ~]$ sudo pacman -S docker
resolving dependencies...
looking for conflicting packages...

Packages (4) containerd-2.1.1-1  libtool-2.5.4+r23+g5b582aed-1  runc-1.3.0-1  docker-1:28.2.0-1

Total Download Size:    46.96 MiB
Total Installed Size:  195.17 MiB

:: Proceed with installation? [Y/n] Y

```




To install the latest minikube stable release on x86-64 Linux using binary download:
```bash
$ curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

Install kubectl

```bash
arch@archlinux ~]$ sudo pacman -S kubectl
resolving dependencies...
looking for conflicting packages...

Packages (1) kubectl-1.33.1-1

Total Download Size:   16.79 MiB
Total Installed Size:  87.55 MiB

:: Proceed with installation? [Y/n] Y
```


From a terminal with administrator access (logged in as arch on the guest)

```bash
minikube start
```

