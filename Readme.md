# Rocker <img width="38" src="https://camo.githubusercontent.com/cd082848a3f8f00d426e36a8fe232a8d44b0e9c6/68747470733a2f2f656d6f6a692e736c61636b2d656467652e636f6d2f5430434151303054552f726f636b6f75742f326631653833663338623161643435392e676966" />

## Deprecation Notice

**This repository is deprecated. As of Docker 18.09, [connecting to Docker
daemon over SSH is built-in][builtin], so this helper script is no longer
necessary.**

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
 1. To avoid getting a `Password:` prompt every time you invoke `docker`, you
    must add to the end of your `/etc/sudoers` file:
    ```
    # Make `sudo -E rockerd` work without a password
    YOUR_USERNAME ALL=(root) NOPASSWD:SETENV: /usr/local/bin/rockerd
    ```
    Be sure to replace `YOUR_USERNAME` with your actual username, and update the
    file location if you installed it to a non-default location.

### Server

Instructions are for an Ubuntu server:

 1. [Setup `docker`](https://docs.docker.com/install/linux/docker-ce/ubuntu).
 1. [Setup SSH server](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring).
 1. Install `sshfs`:
    ```bash
    $ sudo apt-get install sshfs
    ```
 1. Create `/mnt/sshfs` with full permissions for the user you will log in as.
    ```bash
    $ sudo mkdir -p /mnt/sshfs
    $ sudo chmod 777 /mnt/sshfs
    ```


## Sub-commands

All `rockerd` commands expect either the `DOCKER_HOST` environment variable to be
set, or for the `--host`/`-H` CLI argument to be specified.

### SSH into the remote docker machine

```bash
$ rockerd ssh
```

Invokes `ssh` with the configured docker host. The ssh connection uses the
control socket that the `rockerd` daemon creates, so it does not need to perform
a new handshake, etc.

### Explicitly forwarding a port to localhost

Publish port 80 from the remote docker machine to localhost:

```bash
$ rockerd port publish 80
```

When using `docker run --net=host`, the ports that the container binds to are
_not known_. In this case, it can be useful to manually publish the desired port
by running the `rockerd port publish` command.

When you no longer want to have the port being forwarded, use `unpublish`:

```bash
$ rockerd port unpublish 80
```


[builtin]: https://medium.com/lucjuggery/docker-tips-access-the-docker-daemon-via-ssh-97cd6b44a53
