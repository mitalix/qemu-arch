
I installed archlinux into qemu successfully (I could install from the network), but would not boot up. So, I downloaded a premade archlinux virtual machine and booted up on qemu. Using user network and host translation I can ssh into the running image. Even while I have set the variables
```bash
http_proxy=proxy.company.com:1234
https_proxy=proxy.company.com:1234
```
I am not able to upgrade (pacman -Syu). I get some kind of TLS timeout errors.

It sounds like you're encountering TLS timeout errors when trying to run pacman -Syu on your Arch Linux virtual machine in QEMU, likely due to network proxy issues. Since you're using a company proxy (proxy.company.com:1234) and user-mode networking with host translation in QEMU, let's troubleshoot and resolve this step-by-step.
#### Possible Causes
- Proxy Configuration: The http_proxy and https_proxy environment variables may not be correctly configured or propagated to pacman.

- QEMU User-Mode Networking: QEMU's user-mode networking (SLiRP) may not fully support proxy settings or may have connectivity limitations.

- TLS/SSL Issues: The TLS timeout could be due to certificate issues, proxy interception, or network restrictions.

- Pacman Configuration: Pacman may not be honoring the proxy settings or may require additional configuration.

- Firewall/Proxy Restrictions: Your company's proxy might be blocking or timing out certain connections.

- Steps to Resolve
Verify Proxy Environment Variables
Ensure the proxy variables are correctly set and accessible to pacman. Run the following in the terminal where you execute pacman:
bash

```bash
echo $http_proxy
echo $https_proxy
```

If they are not set or incorrect, set them again:
bash

```bash
export http_proxy="http://proxy.company.com:1234"
export https_proxy="http://proxy.company.com:1234"
```

To make these persistent across sessions, add them to your shell configuration (e.g., ~/.bashrc or /etc/environment for global settings):
bash

```bash
echo 'export http_proxy=http://proxy.company.com:1234' >> ~/.bashrc
echo 'export https_proxy=http://proxy.company.com:1234' >> ~/.bashrc
source ~/.bashrc
```
2. Configure Pacman to Use the Proxy
Pacman itself may not automatically use the http_proxy and https_proxy environment variables. You can configure a proxy in /etc/pacman.conf or use a tool like wget (which pacman relies on for downloads) to respect proxy settings.
Check if wget is honoring the proxy by testing a download:
bash
```bash
wget -S https://archlinux.org
```
If wget fails with a TLS timeout, it confirms the proxy or network issue. Configure wget to use the proxy by editing ~/.wgetrc or /etc/wgetrc:
bash
```bash
echo 'use_proxy = on' >> ~/.wgetrc
echo 'http_proxy = http://proxy.company.com:1234' >> ~/.wgetrc
echo 'https_proxy = http://proxy.company.com:1234' >> ~/.wgetrc
```
Then retry pacman -Syu.
3. Check QEMU Networking
QEMU's user-mode networking (SLiRP) is simple but limited and may not handle complex proxy setups well. Since you can SSH into the VM, basic networking is working, but HTTPS traffic may be restricted.
Test Connectivity: Verify if you can reach external servers without HTTPS (to isolate TLS issues):

bash

ping 8.8.8.8
curl http://archlinux.org

Switch to TAP Networking (Optional): If user-mode networking is causing issues, consider switching to TAP networking for better control. This requires root privileges on the host and setting up a bridge:
bash

##### On the host, create a TAP interface
sudo ip tuntap add dev tap0 mode tap
sudo ip link set tap0 up
sudo ip link set tap0 master br0  # Assuming a bridge 'br0' exists

##### Update QEMU command to use TAP
qemu-system-x86_64 -net nic -net tap,ifname=tap0,script=no,downscript=no ...

Then configure the guest to use DHCP or a static IP. This may bypass some user-mode networking limitations.

4. Inspect TLS Timeout Errors
The TLS timeout could be due to:
Proxy SSL Inspection: Your company proxy might be intercepting HTTPS traffic, causing certificate mismatches. Check if the proxy provides a custom CA certificate that needs to be installed:
bash

##### Add the proxy's CA certificate to the system
sudo cp company-ca-cert.pem /etc/ca-certificates/trust-source/anchors/
sudo update-ca-certificates

Obtain the company-ca-cert.pem from your IT department.

Clock Skew: Ensure the VM's clock is synchronized, as incorrect time can cause TLS errors:
bash

sudo timedatectl set-ntp true
timedatectl status

Test with curl: Use curl to debug TLS issues:
bash

curl -v https://mirror.archlinux.org

Look for errors related to certificates, timeouts, or proxy issues in the output.

5. Update Pacman Mirrorlist
If the issue is specific to certain mirrors, refresh the mirrorlist to use more reliable or geographically closer mirrors:
bash

sudo pacman -Syy  # Refresh mirrorlist

Alternatively, manually edit /etc/pacman.d/mirrorlist to prioritize mirrors that work better through your proxy. You can generate a new mirrorlist using:
bash

curl -s 'https://archlinux.org/mirrorlist/?country=all&protocol=https&use_mirror_status=on' | sudo tee /etc/pacman.d/mirrorlist

6. Bypass Proxy for Testing
If possible, test without the proxy to isolate the issue. Temporarily unset the proxy variables:
bash
```bash
unset http_proxy
unset https_proxy
sudo pacman -Syu
```
If this works, the proxy is likely the issue. Contact your IT department to confirm the proxy allows access to Arch Linux mirrors (e.g., https://mirror.archlinux.org).
7. Debug with Verbose Output
Run pacman with verbose output to get more details on the TLS timeout:
bash

sudo pacman -Syu --debug

Check the output for specific error messages, such as which mirror is failing or details about the TLS handshake.
8. Alternative: Use a Different Package Manager or Tool
If pacman continues to fail, try using an alternative like yay or paru (AUR helpers) to see if they handle the proxy better, or use curl/wget to manually download packages and install them with pacman -U.
9. Check Company Proxy Logs
If none of the above work, the issue may be on the proxy server side. Contact your IT department to check if:
The proxy is blocking Arch Linux mirror domains.

HTTPS traffic is being throttled or timing out.

Specific ports or protocols required by pacman are restricted.

Example QEMU Command for Reference
Ensure your QEMU command is correctly configured for networking. A typical user-mode networking setup looks like:
bash

qemu-system-x86_64 -m 2G -drive file=archlinux.qcow2,format=qcow2 -net nic -net user,hostfwd=tcp::2222-:22

Here, -net user,hostfwd=tcp::2222-:22 forwards port 2222 on the host to port 22 (SSH) on the guest, allowing SSH access. Verify this setup matches your configuration.
Next Steps
Try the above steps in order, starting with verifying proxy settings and testing with wget or curl.

If the issue persists, share the exact TLS timeout error message or output from pacman -Syu --debug for more specific guidance.

If you suspect QEMU networking is the bottleneck, consider experimenting with TAP networking or consulting QEMU documentation for advanced networking options.

Let me know if you need help with any specific step or additional details!

QEMU networking options

Arch Linux mirrors

