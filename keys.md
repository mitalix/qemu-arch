I installed archlinux into qemu successfully (I could install from the network), but would not boot up. So, I downloaded a premade archlinux virtual machine and booted up on qemu. Using user network and host translation I can ssh into the running image. Even while I have set the variableshttp_proxy=proxy.company.com:1234
https_proxy=proxy.company.com:1234
I am not able to upgrade (pacman -Syu). I get some kind of TLS timeout errors.I could run curl --proxy proxy.company.com:1234  https://mirror.archlinux.org on that archlinux virtual machine successful on that corporate network. So, it must be something about pacman or overall key management within archlinux.

The issue you're encountering with pacman -Syu failing due to TLS timeout errors, despite successfully using curl with the corporate proxy, suggests a problem specific to pacman's configuration or its handling of the proxy and TLS settings. Since you can use curl to access the Arch Linux mirror through the proxy, the network and proxy settings are likely functional, and the issue lies with pacman or the system's TLS/keyring configuration. Here’s a step-by-step approach to troubleshoot and resolve the issue:
1. Verify Proxy Settings for pacman
Environment Variables: You’ve set http_proxy and https_proxy environment variables, but ensure they are correctly applied for pacman. These variables need to be available in the environment where pacman runs, especially if you’re running it with sudo or as root, since sudo may not preserve environment variables.
Run pacman with the environment variables explicitly:
bash

sudo env http_proxy=http://proxy.company.com:1234 https_proxy=http://proxy.company.com:1234 pacman -Syu

Alternatively, set the variables globally in /etc/environment or /etc/profile to ensure they persist:
bash

echo 'export http_proxy=http://proxy.company.com:1234' | sudo tee -a /etc/environment
echo 'export https_proxy=http://proxy.company.com:1234' | sudo tee -a /etc/environment
source /etc/environment

Pacman Configuration: Check if pacman is configured to use the proxy. Edit /etc/pacman.conf and ensure no conflicting proxy settings are present. You can also explicitly set the proxy in pacman.conf by adding:
bash

XferCommand = /usr/bin/wget --proxy=http://proxy.company.com:1234 %u

This forces pacman to use wget with the specified proxy. Alternatively, if curl is preferred:
bash

XferCommand = /usr/bin/curl --proxy http://proxy.company.com:1234 -L -o %o %u

2. Check TLS and Certificate Issues
TLS Timeout Errors: TLS timeouts can occur if pacman is unable to negotiate a secure connection with the mirror due to certificate issues, outdated TLS libraries, or proxy interference.
Update the Arch Linux Keyring: Ensure the keyring is up-to-date, as outdated or missing keys can cause TLS issues:
bash

sudo pacman -S archlinux-keyring

If this command fails due to the same TLS error, try manually downloading and installing the keyring package:
bash

curl --proxy http://proxy.company.com:1234 -O https://mirror.archlinux.org/core/os/x86_64/archlinux-keyring-<version>.pkg.tar.zst
sudo pacman -U archlinux-keyring-<version>.pkg.tar.zst

Replace <version> with the latest version from the mirror.

Initialize GnuPG: Ensure the GnuPG keyring is initialized:
bash

sudo pacman-key --init
sudo pacman-key --populate archlinux

Corporate Proxy Certificates: If your corporate proxy uses a custom SSL certificate (e.g., for HTTPS inspection), pacman may not trust it. You need to add the proxy’s CA certificate to the system’s trust store:
Obtain the proxy’s CA certificate (ask your network admin or export it from a browser).

Place it in /etc/ssl/certs/ or /usr/local/share/ca-certificates/:
bash

sudo cp proxy-ca-cert.pem /usr/local/share/ca-certificates/
sudo update-ca-certificates

Alternatively, tell pacman to use curl with a specific CA bundle:
bash

XferCommand = /usr/bin/curl --proxy http://proxy.company.com:1234 --cacert /path/to/proxy-ca-cert.pem -L -o %o %u

TLS Protocol Issues: Some proxies or mirrors may require specific TLS versions. Test with curl to force TLS 1.2 or 1.3:
bash

curl --proxy http://proxy.company.com:1234 --tlsv1.2 https://mirror.archlinux.org

If this works, configure pacman to use curl with the same TLS version by modifying the XferCommand in /etc/pacman.conf.

3. Mirror Configuration
Check Mirror Status: The mirror you’re using (https://mirror.archlinux.org) may be slow or have issues. Test other mirrors:
Edit /etc/pacman.d/mirrorlist and prioritize a different mirror, e.g.:
bash

Server = https://mirrors.kernel.org/archlinux/$repo/os/$arch

Use reflector to automatically select fast mirrors (if installed):
bash

sudo reflector --latest 10 --sort rate --save /etc/pacman.d/mirrorlist

Force HTTP Temporarily: If HTTPS continues to fail, test with an HTTP mirror (less secure but useful for debugging):
bash

Server = http://mirror.archlinux.org/$repo/os/$arch

4. Debugging pacman
Verbose Output: Run pacman with debug output to get more details on the TLS timeout:
bash

sudo pacman -Syu --debug

Look for specific errors related to the proxy, TLS handshake, or certificate validation.

Check Network Connectivity: Ensure the VM’s network configuration in QEMU (user networking with host translation) isn’t causing issues. Verify DNS resolution:
bash

ping mirror.archlinux.org
nslookup mirror.archlinux.org

If DNS fails, set a public DNS server in /etc/resolv.conf:
bash

echo 'nameserver 8.8.8.8' | sudo tee /etc/resolv.conf

5. Compare with curl
Since curl works, compare its behavior with pacman. Run curl with verbose output to see the exact TLS negotiation:
bash

curl --proxy http://proxy.company.com:1234 -v https://mirror.archlinux.org

Note the TLS version, cipher suite, and certificate chain. If curl negotiates successfully but pacman doesn’t, it’s likely a pacman-specific issue with the downloader (libalpm uses libcurl internally but may have different defaults).

6. Workaround: Manual Package Download
If pacman continues to fail, use curl to download packages manually and install them:
bash

curl --proxy http://proxy.company.com:1234 -O https://mirror.archlinux.org/core/os/x86_64/<package>.pkg.tar.zst
sudo pacman -U <package>.pkg.tar.zst

This bypasses pacman’s internal downloader for updates.

7. QEMU-Specific Considerations
Clock Synchronization: TLS issues can arise if the VM’s clock is out of sync, as certificates rely on correct time. Ensure the system time is correct:
bash

sudo timedatectl set-ntp true
timedatectl status

Networking Mode: User-mode networking in QEMU may introduce quirks. If possible, test with a different networking mode (e.g., TAP or bridged networking) to rule out issues with QEMU’s NAT or port forwarding.

8. Premade VM Considerations
Since you’re using a premade Arch Linux VM, it may have outdated packages or misconfigured settings. Ensure the system is fully updated:
bash

sudo pacman -Syyu

If this fails, try updating individual components like curl, openssl, and gnupg manually using the method in step 6.

Check if the premade VM has any custom configurations in /etc/pacman.conf or /etc/pacman.d/ that might interfere with your proxy settings.

9. Corporate Network Specifics
Proxy Authentication: If your corporate proxy requires authentication, ensure you include credentials in the proxy URL:
bash

export http_proxy=http://username:password@proxy.company.com:1234
export https_proxy=http://username:password@proxy.company.com:1234

Proxy Restrictions: Some corporate proxies block specific TLS protocols or ciphers. Work with your network admin to confirm the proxy supports the TLS versions and ciphers used by pacman’s downloader (libcurl).

Example Workflow
Set proxy environment variables:
bash

export http_proxy=http://proxy.company.com:1234
export https_proxy=http://proxy.company.com:1234

Update the keyring:
bash

sudo pacman -S archlinux-keyring

Configure pacman to use curl with the proxy:
bash

echo 'XferCommand = /usr/bin/curl --proxy http://proxy.company.com:1234 -L -o %o %u' | sudo tee -a /etc/pacman.conf

Sync and update:
bash

sudo pacman -Syu

If the Issue Persists
Share the exact TLS timeout error message from pacman --debug for more specific guidance.

Test with a different mirror or HTTP (non-TLS) to isolate whether the issue is TLS-specific.

If your corporate network is highly restrictive, consult with your network admin to ensure the proxy allows pacman’s traffic.

This approach should help you diagnose and resolve the TLS timeout issue with pacman on your Arch Linux VM.

learn about libalpm

troubleshoot qemu networking

