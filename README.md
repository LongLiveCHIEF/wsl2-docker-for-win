# WSL2 Docker on Windows

  - [Prerequisites](#prerequisites)
  - [Usage](#usage)
  - [Installation](#installation)

Back in July, Docker announnced that with WSL2, they could create a more native Linux
experience for Docker on Windows, and in so doing, opened up the door for that experience
to work on Windows 10 Home.

Docker _does_ have a [wsl2 tech preview], but as of the time of this writing, you 
[can't install it on Windows 10 Home][issue-4586].

Well, I got impatient, and started tinkering, and wouldn't you know, I got it working.

This repository will start as a document of the steps I took, and hopefully evolve into
a simple script to allow automation of this setup.  

## Prerequisites

- Be running Windows Build 18945 or higher. (`18917` or higher shoud work, but you won't get automatic localhost resolution between Windows and Linux processes)
- Have wsl2 setup [Microsoft Docs - Install WSL2][install-wsl2]
- Ubuntu 18.04 installed [Store link - Ubuntu 18.04][ubuntu-store]
- (Optional) Install new Windows Terminal [from Store][windows-terminal-store]
- (Optional) Install [chocolatey][]

## Usage

So here's the thing. Once I got everything up and runing, I found I had absolutely no need
to install the native windows docker client!  I wound up mapping `\\wsl$\\Ubuntu-18.04` to 
my `U:` drive, and automounting  `root = /` in `/etc/wsl.conf` on Ubuntu (Installation step 1).

As a result, I can now navigate to `U:` in windows explorer or my editor of choice to open and edit
files in the Ubuntu filesystem, _including_ files on any mapped Windows drive.

For example, I have a 240G SSD mounted as `Y` drive, and that is exclusively where I have my
source code.

Now, I can open VSCode on Windows, and create a new project at `Y:/example_project`, and those files
will be accessible _inside Ubuntu_ at `/y/example_project`. 

In VSCode I set my default terminal as the Ubuntu bash terminal, (require the new [Windows Terminal][windows-terminal-store]), and `cd /y/example_project`.

I can now edit files on windows _or_ on Linux, and use `docker run -v $PWD:/app` and it will mount
the files I'm editing in windows, into the docker container that is running on Linux.
The same concept works with _any_ Windows drive.
I could have created my project at `C:\Users\username\example_project` and `cd /c/Users/username/example_project`
in my terminal.

This _is_ the same experience you get with docker on its native platform (Linux), _without_ having to use
some tiny VM emulation layer, and having to deal with samba mounts and [passwords to share][c-share-passwords] 
your `C` drive for the
windows docker client. WSL gives you seamless integration with Ubuntu, including [exposing ports on
localhost between the two kernels][localhost-mapping-wsl].

Therefore, installing the docker client on Windows is _completely_ optional! (I included the steps below anyways,
since you might have your own reasons for wanting low-level powershell docker integration)

## Installation

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

1. Download the docker client from the following url, substituting for the release you want:

```
https://dockermsft.blob.core.windows.net/dockercontainer/docker-19-03-1.zip
```

2. Unzip the contents of the `docker-19.03-1.zip` to `C:\Program Files\Docker` (new directory)
*Note:* Make sure that the `docker.exe` is at `C:\Program files\Docker\docker.exe`

3. Add `C:\Program Files\Docker` to the _System_ `PATH` Environment Variable
4. Run `refreshenv` in Powershell and you should now be able to run `docker version`!

You should see output similar to below:

```
Client: Docker Engine - Community
 Version:           19.03.1      
 API version:       1.40    
 Go version:        go1.12.5
 Git commit:        74b1e89 
 Built:             Thu Jul 25 21:21:05 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.1
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.5
  Git commit:       74b1e89
  Built:            Thu Jul 25 21:19:41 2019
  OS/Arch:          linux/amd64
  Experimental:     true
 containerd:
  Version:          1.2.6
  GitCommit:        894b81a4b802e4eb2a91d1ce216b8817763c29fb
 runc:
  Version:          1.0.0-rc8
  GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

### Install docker-compose on Windows

Using [chocolatey][] is the easiest for this one:

```
choco install docker-compose
```

[c-share-passwords]: https://github.com/docker/for-win/issues/616 
[chocolatey]: https://chocolatey.org/
[Docker Installing on Ubuntu]: https://docs.docker.com/install/linux/docker-ce/ubuntu/
[install-wsl2]: https://docs.microsoft.com/en-us/windows/wsl/wsl2-install
[issue-4586]: https://github.com/docker/for-win/issues/4586
[localhost-mapping-wsl]: https://devblogs.microsoft.com/commandline/whats-new-for-wsl-in-insiders-preview-build-18945/
[ubuntu-store]: https://www.microsoft.com/store/productId/9N9TNGVNDL3Q
[windows-terminal-store]: https://www.microsoft.com/store/productId/9N0DX20HK701
[wsl2 tech preview]: https://docs.docker.com/docker-for-windows/wsl-tech-preview/