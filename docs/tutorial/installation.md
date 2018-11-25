---
title: Installation
parent: Tutorial
permalink: /tutorial/installation
nav_order: 1
---
# Installation
{:.no_toc}

This section will cover how to install Docker and Docker Compose.

Although Docker can be installed on any platform, I would recommend following this tutorial on a Linux distribution as there are known networking limitations on Windows/MacOS. As such, this tutorial is written for Ubuntu 18.04, but previous LTS versions should also work.  

---
## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Setup Repo
1. Update the `apt` package index:
```
sudo apt-get update
```

2. Install packages to allow `apt` to use a repository over HTTPS:
```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

3. Add Docker’s official GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4. Add the stable repository:
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

---
## Install Docker-CE
1. Update the apt package index:
```
sudo apt-get update
```

2. Install the latest version of Docker CE:
```
sudo apt-get install docker-ce
```

3. Add user to docker group:
```
sudo usermod -aG docker <user>
```

4. Log out to apply changes

---
## Install Docker Compose
1. Download the latest version of Docker Compose:
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2. Apply executable permissions to the binary:
```
sudo chmod +x /usr/local/bin/docker-compose
```

3. Add bash completion:
```
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.23.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```

---
## Issues

### Permission Error
Example:  
```
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
```
Solution:  
Add your user to the docker group _and_ log out for the changes to apply.

### Package Issue
Example:  
```
E: Package 'docker-ce' has no installation candidate
```
Solution:  
You are likely not using an LTS-version of Ubuntu.  

Find the file `/etc/apt/sources.list` and edit the appropriate line:  
```
--- deb [arch=amd64] https://download.docker.com/linux/ubuntu cosmic stable
+++ deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
```

---
## References
{:.no_toc}
<https://docs.docker.com/install/linux/docker-ce/ubuntu/>  
<https://docs.docker.com/compose/install/>  
<https://docs.docker.com/compose/completion/>  
