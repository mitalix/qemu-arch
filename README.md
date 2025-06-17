# qemu-arch

---

- Install qemu on host machine
- Install or acquire VM image for guest machine
- Kubernetes
    - docker
    - minikube (requires at least 2G memory)
    - kubectl
    - kubelet



https://wiki.archlinux.org/title/Installation_guide

Presently, the best option for archlinux on qemu is a virtual machine image https://gitlab.archlinux.org/archlinux/arch-boxes/-/packages


##### Configuration Start

```bash
image_dir=../images/archlinux
disk=${image_dir}/Arch-Linux-x86_64-basic-20250615.366044.qcow2
```

Using the virtual machine image is more convenient, because there is no lengthy installation process
```bash
$ qemu-system-x86_64 -smp 6 -m 2G -hda ${disk}
```
Run in another window to see the screen:
```bash
$ vncviewer :5900
```
Make sure openssh is installed on the host, if you want to ssh to the guest user network
```bash
$ qemu-system-x86_64 -smp 6 -m 4G -hda ${disk} -net nic -net user,hostfwd=tcp::2222-:22
```


When the network works fine, do an upgrade

```bash
[arch@archlinux ~]$ sudo su pacman -Syu
```


Install docker, the dependency  containerd is included automatically

```bash
[arch@archlinux ~]$ sudo pacman -S docker
resolving dependencies...
looking for conflicting packages...

Packages (4) containerd-2.1.1-1  libtool-2.5.4+r23+g5b582aed-1  runc-1.3.0-1  docker-1:28.2.0-1

Total Download Size:    46.96 MiB
Total Installed Size:  195.17 MiB

:: Proceed with installation? [Y/n] Y
```
https://kubernetes.io/docs/tutorials/hello-minikube/

Install the latest minikube stable release on x86-64 Linux using binary download:
```bash
[arch@archlinux ~]$ curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
[arch@archlinux ~]$ sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

Install kubectl and the node agent kubelet

- kubectl - A command line tool for communicating with a Kubernetes API server

- kubelet - An agent that runs on each node in a Kubernetes cluster making sure that containers are running in a Pod


```bash
[arch@archlinux ~]$ sudo pacman -S kubectl
[arch@archlinux ~]$ sudo pacman -S kubelet
```


Enable and start docker and kubelet, containerd will start automatically.
```
[arch@archlinux ~]$ sudo systemctl enable --now docker
Created symlink '/etc/systemd/system/multi-user.target.wants/docker.service' → '/usr/lib/systemd/system/docker.service'.

arch@archlinux ~]$ sudo systemctl enable --now kubelet
Created symlink '/etc/systemd/system/multi-user.target.wants/kubelet.service' → '/usr/lib/systemd/system/kubelet.service'.
```

Give it a reboot, or not
```bash
[arch@archlinux ~]$ sudo reboot
```
Health check
```bash
[arch@archlinux ~]$ echo;list="docker kubelet containerd"; for service in ${list}; do echo -en Svc ${service}:"\t"; echo -en $(systemctl is-enabled ${service})' \t'$(systemctl is-active ${service})'  \t'failure status : $(systemctl is-failed ${service});echo;done; echo System-wide status: $(systemctl  is-system-running)

Svc docker: 	enabled 	active  	failure status : active
Svc kubelet:	enabled 	active  	failure status : active
Svc containerd:	disabled 	active  	failure status : active
System-wide status: running
```


Install your favorite editor

```bash
[arch@archlinux ~]$ sudo pacman -S vim
```
Sudo is not necessary, when you add the user to the docker group
```bash
[arch@archlinux ~]$ sudo vim /etc/group
[arch@archlinux ~]$ grep docker /etc/group
docker:x:971:arch
```


Test docker with a hello-world

```bash
[arch@archlinux ~]$ docker run hello-world
```
##### Configuration End

##### Operational State Start

> [!WARNING]
Disable swap[^1] before using kubelet.service.

```bash
[arch@archlinux ~]$ swapon -s
Filename				Type		Size		Used		Priority
/swap/swapfile                          file		524284		256		-2
[arch@archlinux ~]$ sudo swapoff -a
[arch@archlinux ~]$ swapon -s
[arch@archlinux ~]$ 
```


Crank up minikube, it could take a while
```bash
[arch@archlinux ~]$ minikube start
```
Health checks:

```bash
[arch@archlinux ~]$ echo;list="docker kubelet containerd"; for service in ${list}; do echo -en Svc ${service}:"\t"; echo -en $(systemctl is-enabled ${service})' \t'$(systemctl is-active ${service})'  \t'failure status : $(systemctl is-failed ${service});echo;done; echo System-wide status: $(systemctl  is-system-running)

Svc docker: 	enabled 	active  	failure status : active
Svc kubelet:	enabled 	active  	failure status : active
Svc containerd:	disabled 	active  	failure status : active
System-wide status: running
```


Basic kubernetes healthcheck
```bash
[arch@archlinux ~]$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3m20s
```


All namespaces healthcheck
```bash
[arch@archlinux ~]$ kubectl get all --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS        AGE
kube-system   pod/etcd-minikube                      1/1     Running   2 (10m ago)     103m
kube-system   pod/kube-apiserver-minikube            1/1     Running   5 (10m ago)     103m
kube-system   pod/kube-controller-manager-minikube   1/1     Running   9 (5m18s ago)   103m
kube-system   pod/kube-scheduler-minikube            1/1     Running   2 (10m ago)     103m

NAMESPACE   NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
default     service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   62m
```


---

#### Optional Network configuration

The username/password default  combination for the base VM is arch/arch. Log into the system from the host:
```bash
$ ssh -p 2222 arch@localhost
```
You could automatically login when you copy your public key from the host to the guest virtual machine
```bash
$ ssh-copy-id -p 2222 arch@localhost
$ ssh -p 2222 arch@localhost
Last login: Sat Jun  7 21:42:09 2025 from 10.0.2.2
[arch@archlinux ~]$ 
```
- Sftp works

```bash
$ sftp arch@localhost -oPort=2222
```
- User mode virtio drivers. Using para-virtualized drivers and configured user networking with port forwarding to expose SSH on the host machine

```bash
qemu-system-x86_64 -m 4G -drive file=${disk},if=virtio -netdev user,id=net0,hostfwd=tcp::2222-:22 -device virtio-net-pci,netdev=net0
```
- Ssh via vsocks.  Try using without needing to configure a network interface within the guest. Also, try with kmv accelerator.
```bash
$ qemu-system-x86_64 -accel kvm -smp 4 -m 4G -hda ${disk} -device vhost-vsock-pci,id=vhost-vsock-pci0,guest-cid=555 
```
```bash
[italix@flintstone New]$ ssh arch@vsock/555
Warning: Permanently added 'vsock/555' (ED25519) to the list of known hosts.
Last login: Sun Jun 15 08:09:34 2025 from UNKNOWN
[arch@archlinux ~]$ 
```


---
Notes:
[^1]: You could completely disable swap by commenting out the swap section in /etc/fstab