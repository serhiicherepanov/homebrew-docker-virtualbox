# Homebrew Docker Virtualbox (Not Only for VirtualBox)

This Homebrew formula fixes Docker issues on AMD‑based macOS systems (Ryzentosh), and it also works on regular Macs using VirtualBox or other Docker Machine drivers.

**Before starting:** read this notice: [https://gist.github.com/slykar/e92732be9bf81a71e08068245656d70e?permalink_comment_id=4105556#gistcomment-4105556](https://gist.github.com/slykar/e92732be9bf81a71e08068245656d70e?permalink_comment_id=4105556#gistcomment-4105556)

---

## 1. Install VirtualBox (Optional)

Download VirtualBox: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

> Required only if you want to use the VirtualBox driver. Remove previous installations first.

---

## 2. VirtualBox 6.1.28+ Network Configuration

If you use VirtualBox, allow access to 192.168.99.0/8:

Edit:

```
sudo nano /etc/vbox/networks.conf
```

Add:

```
* 192.168.99.0/8
```

---

## 3. Install docker‑virtualbox via Homebrew

```
brew tap serhiicherepanov/docker-virtualbox
brew install docker-virtualbox
```

---

## 4. Configure docker‑virtualbox Requirements

These require root permissions:

Ensure NFS exports file exists:

```
sudo touch /etc/exports
```

Allow the `staff` group necessary permissions:

```
sudo tee /etc/sudoers.d/docker-machine-nfs <<SUDOERS
%staff ALL=(ALL) NOPASSWD: /sbin/nfsd
%staff ALL=(ALL) NOPASSWD: /bin/cp /etc/nfs.conf /etc/nfs.conf.bak
%staff ALL=(ALL) NOPASSWD: /usr/bin/tee /etc/exports
%staff ALL=(ALL) NOPASSWD: /usr/bin/tee /etc/nfs.conf
%staff ALL=(ALL) NOPASSWD: $(brew --prefix)/opt/docker-virtualbox/bin/gobetween
SUDOERS
```

> Reboot to apply changes.

---

## 5. Configure Environment

If **not** using Docker Desktop:

```
brew link --force --overwrite docker-virtualbox
```

If **using** Docker Desktop:

**bash**

```
echo "export PATH=\"$(brew --prefix docker-virtualbox)/bin:$PATH\"" >> ~/.bash_profile
```

**zsh**

```
echo "export PATH=\"$(brew --prefix docker-virtualbox)/bin:$PATH\"" >> ~/.zshrc
```

Reload:

```
exec $SHELL
```

---

## 6. Initialize Docker Machine

Run once:

```
docker-machine-init initialize
```

This will download, prepare, and configure the VirtualBox Docker Machine.

---

## 7. Start docker‑virtualbox

```
brew services start docker-virtualbox
```

Logs:

```
/tmp/docker-virtualbox.log
```

---

## 8. Test Docker

```
docker run -d -p 8989:80 nginx
curl -v localhost:8989
```

---

# Additional Options

## Use a Non‑VirtualBox Driver

Example (generic SSH driver):

```
docker-machine create \
  --driver generic \
  --generic-ip-address=192.168.24.108 \
  --generic-ssh-user=developer \
  --generic-ssh-key=$HOME/.ssh/id_rsa \
  docker
```

> Works only with Debian‑based remote systems.

---

## Useful Commands

View logs:

```
tail -n 1000 -f /tmp/docker-virtualbox.log
```

SSH into the machine:

```
docker-machine ssh docker
```

Stop service:

```
brew services stop docker-virtualbox
```

Load environment for tools:

```
source /tmp/docker-virtualbox.env
source /tmp/docker-virtualbox-machine.env
```

---

# Optional Enhancements

## Auto‑Start on Login

Create launch agents:

```
mkdir -p ~/Library/LaunchAgents
```

Configure a custom plist to auto‑start docker‑virtualbox.

## Increase VM Resources

```
docker-machine stop docker
VBoxManage modifyvm docker --memory 4096 --cpus 4
docker-machine start docker
```

## Custom Network Range

Edit:

```
sudo nano /etc/vbox/networks.conf
```

Add:

```
* 10.10.0.0/16
```

---

# Known Issues

1. macOS cannot sleep while NFS is running.
2. UDP port forwarding not supported.
3. VirtualBox may need kext reload after macOS updates.
4. File sharing may be slower compared to Docker Desktop.
