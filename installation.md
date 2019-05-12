# Installation

## Outline

In this chapter you will learn:

* How to install Docker into your system

## Walkthrough

[Hot fix] Disable apt daily upgrade:

```bash
$ sudo systemctl stop apt-daily-upgrade
$ sudo systemctl disable apt-daily-upgrade
$ sudo systemctl stop apt-daily
$ sudo systemctl disable apt-daily
```

Update the `apt` package index:

```bash
$ sudo apt-get update
```

Install additional packages required for Docker installation and further workshop exercises:

```bash
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    gnupg-agent \
    software-properties-common \
    curl \
    wget \
    vim \
    git-core \
    htop \
    tmux \
    redis-tools
```

Add Docker’s official GPG key

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Setup the stable Docker repository:

```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Update the package index:

```bash
$ sudo apt-get update
```

Install Docker packages:

```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Check the installed Docker version:

```bash
$ sudo docker version
Client: Docker Engine - Community
 Version:           18.09.0
 API version:       1.39
 Go version:        go1.10.4
 Git commit:        4d60db4
 Built:             Wed Nov  7 00:47:43 2018
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.0
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.4
  Git commit:       4d60db4
  Built:            Wed Nov  7 00:55:00 2018
  OS/Arch:          linux/amd64
  Experimental:     true
```

Verify the installation:

```
$ sudo docker run --rm hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:92695bc579f31df7a63da6922075d0666e565ceccad16b59c3374d2cf4e8e50e
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

**Docker is now installed and running.**

Try to run a Docker command without `sudo`:

```bash
$ docker run --rm hello-world
```

Current user is lacking permissions. Add current user to `docker` group:

```bash
sudo usermod -aG docker $USER
```

**Restart your VM.**

Log in, and try to run a Docker command without `sudo`:

```bash
$ docker run --rm hello-world
```
