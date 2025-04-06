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
7. Host the ign file on a web server  
It could be:
 - A regular web server like Apache
 - Python/rust "share a folder temporarily" libraries, python3 http.server for example
 - A file hosted on github.com/gitlab.com/etc. this has the advantage of HTTPS with a  for example https://github.com/pilkch/coreos-home/blob/main/fileserver.network.home.bu (Thanks github!)
8. Download the CoreOS iso and copy it to a USB drive (I downloaded the live DVD iso) https://fedoraproject.org/coreos/download?stream=stable (This could also be hosted on a PXE boot server)
9. Use the ignition file to boot Fedora CoreOS https://docs.fedoraproject.org/en-US/fedora-coreos/bare-metal/#_installing_from_live_iso  
Type this at the prompt in the installer (I like the live DVD because you can double check which drive you are installing to):
```bash
sudo coreos-installer install /dev/sda --ignition-url https://github.com/pilkch/coreos-home/blob/main/fileserver.network.home.bu
```
