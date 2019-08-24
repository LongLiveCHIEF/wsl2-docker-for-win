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

This step is mostly going to follow the [Docker Installing on Ubuntu][] docs, with one
major exception. The current Ubuntu distro from Microsoft doesn't use `systemd`, so
you'll have to start/stop docker with `sudo service docker {start|stop|restart|enable|disable}`.


```
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl \
    gnupg-agent software-properties-common
```
Then add dockers gpg key. 

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
```

Now install the _edge_ docker-ce. (stable will probably work, but I suspect less errors
with WSL2 on windows will happen on edge)

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   edge"
```

### Configure Docker on Ubuntu

In order for the `docker` command on the windows host to run
"natively", we'll need to use the daemon running in WSL to allow
connections from the Windows Host.

Since Windows Build `18945`, services on the WSL machine are automatically exposed
to `localhost` on the Windows side, and vice-versa.

There is probably a startup script that can be run to figure out what the IP of the Windows
Host is from the Linux machine, but I haven't gotten that far yet, so for now, we'll
just set the daemon to listen to `unix://` and _optionally_ `tcp://0.0.0.0:2375`.

If you don't intend to use the `docker` command from powershell or a windows terminal,
and map the wsl as a network drive, you can remote the host entry as noted below to
increase security.

Create the following `/etc/docker/daemon.json` file in WSL machine:

```
{
    "hosts": ["unix://", "tcp://0.0.0.0:2375"], // remove the tcp if you only use docker-cli from Ubuntu
    "experimental": true
}
```

### Enable and start the docker daemon

```
sudo service docker enable
sudo service docker start
```

### Set docker env vars on Windows

Open up your Environment Variables in Windows, and add the following
entries under the System Environment Variables section:

| Env Var | Value | Required/Optional |
| ======= | ===== | ================= |
| `DOCKER_HOST` | `tcp://localhost:2375` | Required |
| `DOCKER_CLI_EXPERIMENTAL` | `enabled` | Optional |
| `DOCKER_API_VERSION | `1.40` | Optional |

### Install docker client on Windows

First download the docker client from the following url, substituting for the release you want:

```
https://dockermsft.blob.core.windows.net/dockercontainer/docker-19-03-1.zip
```


### Install docker-compose on Windows

```
// instructions coming soon
```


[wsl2 tech preview]: https://docs.docker.com/docker-for-windows/wsl-tech-preview/
[issue-4586]: https://github.com/docker/for-win/issues/4586
[install-wsl2]: https://docs.microsoft.com/en-us/windows/wsl/wsl2-install
[Docker Installing on Ubuntu]: https://docs.docker.com/install/linux/docker-ce/ubuntu/
[ubuntu-store]: https://www.microsoft.com/store/productId/9N9TNGVNDL3Q
[windows-terminal-store]: https://www.microsoft.com/store/productId/9N0DX20HK701
[chocolatey]: https://chocolatey.org/