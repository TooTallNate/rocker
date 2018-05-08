# Rocker <img width="38" src="https://camo.githubusercontent.com/cd082848a3f8f00d426e36a8fe232a8d44b0e9c6/68747470733a2f2f656d6f6a692e736c61636b2d656467652e636f6d2f5430434151303054552f726f636b6f75742f326631653833663338623161643435392e676966" style="vertical-align: bottom;" />

**R**emote D**ocker** is a small wrapper around the `docker` client that
extends it with support for securely connecting to a remote `dockerd` daemon
via SSH.

In short, it implements the `ssh://` protocol for the `--host` parameter in the
Docker CLI.

Why? Because I use a MacBook 12" which is essentially a tablet in laptop form,
so offloading the `dockerd` daemon to a more powerful machine makes development
a lot faster. This also allows for an easy way to share running docker
containers between teammates for collaboration, or if you develop on more than
one machine.

Aims to support all docker features, in particular:

 * Port publishing (`-p`) by SSH local port forwarding.
 * Volume mounts (`-v`) by a reverse `sshfs` from the server to local.

## Usage

One-time with `-H`/`--host`:

```bash
$ docker -H ssh://user@myserver.com run --rm -p 80:80 nginx
```

Make it permanent by setting `DOCKER_HOST`:

```bash
$ export DOCKER_HOST=ssh://user@myserver.com
$ docker run --rm -p 80:80 nginx
```

## Setup

### Local

 1. Install the `docker` client (Docker for Mac for MacOS).
 1. Setup an SSH server running locally, for the reverse `sshfs` mount to work.
 1. Add the remote docker server's public key to your local
    `~/.ssh/authorized_keys`.
 1. Run `./install.sh` from this repo to install `/usr/local/bin/docker` and
    friends.
 1. Set the `DOCKER_HOST` env var or use `docker -H` with an `ssh://` protocol to
    invoke "rocker".

### Server

Instructions are for an Ubuntu server:

 1. [Setup `docker`](https://docs.docker.com/install/linux/docker-ce/ubuntu).
 1. [Setup SSH server](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring).
 1. Install `sshfs` - `sudo apt-get install sshfs`.
 1. Create `/mnt/sshfs` with full permissions for the user you will log in as.
 1. Configure `dockerd` to listen on a TCP port by adding `-H tcp://127.0.0.1:2375`
    to the `ExecStart` in `/lib/systemd/system/docker.service`.
