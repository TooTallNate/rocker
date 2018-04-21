# Rocker

**R**emote D_ocker_ is a small collection of shell scripts that extends the
`docker` client with support for securely connecting to a remote `dockerd`
daemon via SSH.

Why? Because I use a MacBook 12" which is essentially a tablet in laptop form,
so using offloading the `dockerd` daemon to a more powerful machine makes
development a lot faster. This also allows for an easy way to share running
docker containers between teammates for collaboration, or if you develop on more
than one machine.

Aims to support all docker features, in particular:

 * Volume mounts (`-v`) by a reverse `sshfs` from the server to local.
 * Port publishing (`-p`) by SSH local port forwarding.

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

 1. [Setup SSH server](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring).
 1. [Setup `docker`](https://docs.docker.com/install/linux/docker-ce/ubuntu).
 1. Configure `dockerd` to listen on a TCP port by adding `-H tcp://127.0.0.1:2375`
    to the `ExecStart` in `/lib/systemd/system/docker.service`.

## Usage

```bash
$ docker -H ssh://myserver.com run --rm -p 80:80 nginx
```
