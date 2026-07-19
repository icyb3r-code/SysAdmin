
#  Docker installation on Fedora

## Prerequisites

**OS requirements**

To install Docker Engine, you need a maintained version of one of the following Fedora versions:

* Fedora 38
* Fedora 39

**Uninstall old versions**

Older versions of Docker went by `docker` or `docker-engine`. Uninstall any such older versions before attempting to install a new version, along with associated dependencies.

```bash
 sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

`dnf` might report that you have none of these packages installed.

Images, containers, volumes, and networks stored in `/var/lib/docker/` aren't automatically removed when you uninstall Docker.

## Installation methods

### **Set up the repository**

Install the `dnf-plugins-core` package (which provides the commands to manage your DNF repositories) and set up the repository.

```bash
 sudo dnf -y install dnf-plugins-core

 sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo

```

To check specific Version 

```bash
dnf list docker-ce --showduplicates | sort -r

sudo dnf -y install docker-ce-VERSION_STRING docker-ce-cli-VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

### **Change the Docker root Dir**

Before we start docker and work on it, the best practice is to create a new mount point that has separate volume or partition on the OS level, then we can tell docker engine to use that mount point to store its data inside that volume, below the steps to have this configuration:

```bash
# Linux - create this file if its note exist
# you can add any path you wish 
sudo cat <<EOF >> /etc/docker/daemon.json
{   
	"data-root": "/u01/var/lib/docker" 
}
EOF 
```


```bash
# windows 
C:\ProgramData\Docker\config\daemon.json

# write inside this file 
{   
	"data-root": "d:\docker" 
}
```
### **Enable and Start Docker Service**

Enable and start docker using `systemd` service command called `systemctl` :

```bash
sudo systemctl enable --now docker

```

### **Verify Docker working**

```bash
sudo docker run hello-world
```