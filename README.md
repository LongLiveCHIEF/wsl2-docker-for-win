# WSL2 Docker on Windows

Back in July, Docker announnced that with WSL2, they could create a more native Linux
experience for Docker on Windows, and in so doing, opened up the door for that experience
to work on Windows 10 Home.

Docker _does_ have a [wsl2 tech preview], but as of the time of this writing, you 
[can't install it on Windows 10 Home][issue-4586].

Well, I got impatient, and started tinkering, and wouldn't you know, I got it working.

This repository will start as a document of the steps I took, and hopefully evolve into
a simple script to allow automation of this setup.  

## Prerequisites

- Be running Windows Build 18945 or higher.
- Have wsl2 setup [Microsoft Docs - Install WSL2][install-wsl2]
- and Ubuntu 18.04 installed [Store link - Ubuntu 18.04][ubuntu-store]
- (Optional) Install new Windows Terminal [from Store][windows-terminal-store]
- (Optional) Install [chocolatey][]

## Setup Steps

1. [Add wsl.conf to Ubuntu](#add-wsl-conf-to-ubuntu)
1. [Install Docker on Ubuntu](#install-docker-on-ubuntu)
2. [Configure Docker on Ubuntu](#configure-docker-on-ubuntu)
3. [Set docker env vars on Windows](#set-docker-env-vars-on-windows)
4. [Install docker client on Windows](#install-docker-client-on-windows)
5. [Install docker-compose on Windows](#install-docker-compose-on-windows)

### Add `wsl.conf` to Ubuntu

Save the following file as `/etc/wsl.conf` on your Ubuntu host:

```
[automount]
root = /
options = "metadata"
```
This will mount all of your mapped drives to a corresponding letter
in the root of your Ubuntu filesystem. This is what will allow you
the "native" experience of editing file on Windows and running them
from your windows _or_ ubuntu terminals.

### Install Docker on Ubuntu

```
// instructions coming soon
```

### Configure Docker on Ubuntu

```
// instructions coming soon
```

### Set docker env vars on Windows

```
// instructions coming soon
```

### Install docker client on Windows

```
// instructions coming soon
```

### Install docker-compose on Windows

```
// instructions coming soon
```


[wsl2 tech preview]: https://docs.docker.com/docker-for-windows/wsl-tech-preview/
[issue-4586]: https://github.com/docker/for-win/issues/4586
[install-wsl2]: https://docs.microsoft.com/en-us/windows/wsl/wsl2-install
[ubuntu-store]: https://www.microsoft.com/store/productId/9N9TNGVNDL3Q
[windows-terminal-store]: https://www.microsoft.com/store/productId/9N0DX20HK701
[chocolatey]: https://chocolatey.org/