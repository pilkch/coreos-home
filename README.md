## Butane Source Files for Use With CoreOS

Usage:
1. Generate a hash for your password:  
```bash
mkpasswd --method=yescrypt
```
Or
```bash
openssl passwd -6 <mypassword>
```
2. Copy the generated password hash to the .bu file.
3. Generate SSH keys with:
```bash
ssh-keygen -t ed25519
```
4. Copy the public key to the .bu file.
5. Install butane:
```bash
sudo dnf install butane
```
6. Compile the butane file to generate an ignition file:
```bash
butane --pretty --strict fileserver.network.home.bu > fileserver.network.home.ign
```
7. Download the CoreOS iso and copy it to a USB drive (I downloaded the live DVD iso) https://fedoraproject.org/coreos/download?stream=stable (This could also be hosted on a PXE boot server)
8. Host the ign file on a web server  
It could be:
 - A regular web server like Apache
 - Python/rust "share a folder temporarily" libraries, python3 http.server for example
 - A file hosted on github.com/gitlab.com/etc. this has the advantage of HTTPS with a root CA signed certificate, for example https://raw.githubusercontent.com/pilkch/coreos-home/refs/heads/main/fileserver.network.home.ign (Thanks github!)
OR
 - Add it to the USB drive
9. Use the ignition file to boot Fedora CoreOS https://docs.fedoraproject.org/en-US/fedora-coreos/bare-metal/#_installing_from_live_iso  
Type this at the prompt in the installer (I like the live DVD because you can double check which drive you are installing to):
```bash
sudo coreos-installer install /dev/sda --ignition-url https://raw.githubusercontent.com/pilkch/coreos-home/refs/heads/main/fileserver.network.home.ign
```
Or, for IoT, edit the boot parameters, add this to the linux line:
```bash
coreos.inst.append=ignition.config.url=https://raw.githubusercontent.com/pilkch/coreos-home/refs/heads/main/fileserver.network.home.ign coreos.inst.append=rd.neednet=1
```
10. Restart
11. Login either directly or via SSH and add python3:
```bash
sudo rpm-ostree install --python3
```

## Atomic Fedora Distributions and Silverblue vs CoreOS

If Silverblue had ignition file support then I would look closer at that.

| OS            | Immutable | Butane/Ignition | Firewalld | Python3 for Ansible | Installation |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Fedora Server/Workstation | **No** | **No** | Yes | Yes | **Verbose, no ignition file support** |
| Fedora Silverblue | Yes | **No** | Yes | Yes? | **Verbose, no ignition file support (Maybe in the future?)** |
| Fedora CoreOS | Yes | Yes, really wants to use an ignition file | **No, WTF???**, can be layered, but that is a terrible default state | Can layer it | Great, ignition file creates the user, adds SSH keys |
| Fedora IoT | Yes | Yes | Yes | Can layer it | **Verbose, even with an ignition file (Not polished, really wants to be a regular live DVD like Fedora Workstation? It could really do with a wizard where you enter the ignition file URL, it is then parsed and prefills the rest of the installer for you), constant audit messages before I've even logged in** |

## Butane/Ignition File Format

Butane is very basic, it can set up storage, user/groups, ssh, and configuration files. But it can't install install rpm-ostree packages. There are some work arounds involving calling a systemd service to do more work after boot, but I really just want to give it a list of packages and have ignition apply the layers for me.  
https://github.com/coreos/butane/issues/81  
https://github.com/coreos/fedora-coreos-tracker/issues/681

For the relatively minor task I wanted to do, bootstrap a NAS file server with a samba share, I was hoping I could make it immutable and do everything in the butane file. Maybe it will be able to soon.
