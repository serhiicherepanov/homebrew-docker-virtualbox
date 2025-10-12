# Homebrew Docker Virtualbox (But not VirtualBox only)
This formula resolves the Docker issue on AMD based MacOS (Ryzentosh). Also can be used on any Mac.

*PLEASE READ FIRST THE NOTICE: https://gist.github.com/slykar/e92732be9bf81a71e08068245656d70e?permalink_comment_id=4105556#gistcomment-4105556*

## Installation

### Install Virtualbox from Oracle website
https://www.virtualbox.org/wiki/Downloads. 
> Please don't forget to remove all previous installations.  
> This step required only if you want to use VirtualBox driver, otherwise see below how to start with another driver. 

### Virtualbox 6.1.28+ Note
Due the changes of virtualbox networking, you need to allow docker-machine host in order to use 192.168.99.0/8 network.  [More Details](https://www.virtualbox.org/manual/ch06.html#network_hostonly)

Add this line into 
`/etc/vbox/networks.conf`
```bash
* 192.168.99.0/8
```

### Install the docker-virtualbox via Homebrew
```bash
brew tap serhiicherepanov/docker-virtualbox
brew install docker-virtualbox
```

### Configure the docker-virtualbox requirements
> WARNING: Only this commands requires root permissions, all next should be run under your user

Ensure the NFS exports file exists
```bash
sudo touch /etc/exports
```

Allow the staff group to configure NFS shares and run the balancer without a password prompt 
```bash
sudo tee /etc/sudoers.d/docker-machine-nfs <<SUDOERS
%staff ALL=(ALL) NOPASSWD: /sbin/nfsd
%staff ALL=(ALL) NOPASSWD: /bin/cp /etc/nfs.conf /etc/nfs.conf.bak
%staff ALL=(ALL) NOPASSWD: /usr/bin/tee /etc/exports
%staff ALL=(ALL) NOPASSWD: /usr/bin/tee /etc/nfs.conf
%staff ALL=(ALL) NOPASSWD: $(brew --prefix)/opt/docker-virtualbox/bin/gobetween
SUDOERS
```

> Reboot your system to be sure that sudoers applied

### Configure the environment

If you didn't install Docker for Mac you can link binaries instead of PATH update
```
brew link --force --overwrite docker-virtualbox
```

Otherwise configure the PATH variable
```bash
# For the bash
echo "export PATH=\"$(brew --prefix docker-virtualbox)/bin:\$PATH\"" >> ~/.bash_profile
# For the zsh
echo "export PATH=\"$(brew --prefix docker-virtualbox)/bin:\$PATH\"" >> ~/.zshrc
```

Reload the shell
```
exec $SHELL
```

### Initialize the docker machine
In the first run according to the permissions policy you need to run it manually and approve permissions.  

```bash
docker-machine-init initialize
```
> This is will download, create and configure the VirtualBox-based machine if the machine was not configured before. If the machine was created manually this will only set up NFS mount.


### Start the docker-virtualbox service
When initialization will be finished you are ready to enable the service
> The log file will be always available in `/tmp/docker-virtualbox.log`. 
```bash
brew services start docker-virtualbox 
```

### Verify installation

Test the Docker by running Nginx
```bash
docker run -d -p 8989:80 nginx
curl -v localhost:8989
```

## Additional information

If you don't want to use the VirtualBox as docker-machine driver you need to create a machine manually before initialization by similar command:
```bash
docker-machine create --driver generic --generic-ip-address=192.168.24.108 --generic-ssh-user=developer --generic-ssh-key=$HOME/.ssh/id_rsa docker
```
> Please note the docker-machine-nfs plugin supports only Debian based Linux as the target system. For more information read the [documentation](https://github.com/sergeycherepanov/docker-machine-nfs/blob/master/README.md)

Read the log when the docker doesn't work properly
```bash
tail -n 1000 -f /tmp/docker-virtualbox.log
```

SSH connection to the docker-machine
```bash
docker-machine ssh docker
```

To stop the service just run
```bash
brew services stop docker-virtualbox 
```

To setup environment for 3rd party tools ([`ctop`](https://github.com/bcicen/ctop) as example)
```
source /tmp/docker-virtualbox.env
source /tmp/docker-virtualbox-machine.env
```

## Known issues
1. The system won't sleep when the NFS server runs
2. The port forwarding doesn't work for UDP proto (improvement needed)
