# Packet Images

This repository contains Dockerfiles that we use as the basis for the OSes that we provision. The Dockerfiles contained here are the source for the official Packet.net managed images, other semi-official images are managed by the community.

### Official Images
- Centos7
- Debian 8
- Debian 9
- FreeBSD (not present)
- Scientific Linux 6
- Ubuntu 14.04
- Ubuntu 16.04
- Ubuntu 17.04 (deprecated)
- Ubuntu 17.10
- Virtuozzo (not present)
- VMWare (not present)
- Windows (not present)

### Semi Official Images
- RancherOS - maintained by [Rancher](https://github.com/rancher)
- Container Linux - maintained by [CoreOS](https://github.com/coreos)
- NixOS - maintained by [@grahamc](https://github.com/grahamc) [NixOS](https://github.com/grahamc/packet-provision-nixos-ipxe)


Within this repo you will find a collection of tools, examples and docs for building OS images to be used on Packet.net baremetal servers.

  - Building images from Dockerfile
  - Converting Docker images for use on physical baremetal servers

TLDR:  Build docker image, save docker image to archive and convert the archive to a rootfs
image. The image can be used on a baremetal physical server with or without docker.

### Overview
Branch layout consists of a primacy "base" branch for each supported operating system distro. See centos\_7-base, ubuntu\_17\_10-base, etc. The base branch contains a Dockerfile (per supported architecture) with a suffienct level of customization (stage 1) to produce a standardized operating system experience across Packet.net's hardware offering. If any hardware specific changes are to be included in a particular image, a separate branch is created for the hardware type / plan the image is customized. Any such hardware specific image is formed by using the base image as the template. For example, if we want to create a new image for ubuntu_17_10-supermachine1, the Dockerfile for this branch will use "FROM ubuntu_17_10-base" as to complete a multi-stage (stage 2) build based off the official Packet base image.

### Dependencies
There is only a small list of deps required to run image builds, but we recommend a dedicated
machine or VM for this purpose simply to keep things isolated. This repo makes use of [git-lfs](https://git-lfs.github.com/) for installation asset storage and serves as the source of truth for images cached downstream. The build script uses read-tree for bringing together the base and hardware specific branches at build time.

 - Docker 1.1.11 and above (older version may work)
 - JQ
 - A linux docker host on top of CentOS7 / Ubuntu 16
 - git-lfs

### Installation
**Using git:**

    git clone git@github.com:packethost/packet-images.git
    sudo cp ./tools/packet-save2image /usr/bin/
    chmod u+x /usr/bin/packet-save2image

**or**

**Using wget**

    sudo wget -O /usr/bin/packet-save2image https://raw.githubusercontent.com/packethost/packet-images/master/tools/packet-save2image
    sudo chmod u+x /usr/bin/packet-*

### Example image build
Here we are walking through an example docker image build, docker image save and conversion.
The branch should contain install assets (image, kernel, initrd, modules) and Dockerfile.
Build tools are intentionally kept separate from operating system branches.
To create a new image you may create an orphan branch (to exclude any pre-existing contents) of the packet-images repo.

    [packet-images]$ git checkout --orphan ubuntu_17_10-supermachine1
    [packet-images]$ git rm --cached -r .
    [packet-images]$ vi Dockerfile
    [packet-images]$ echo "RUN apt-get -y install mlx" >> Dockerfile
    [packet-images]$ docker build -t ubuntu_17_10-supermachine1 . && docker save ubuntu_17_10-supermachine1 > ubuntu_17_10-supermachine1.tar && save2-image < ubuntu_17_10-supermachine1 > image.tar.gz
    [packet-images]$ git lfs track *.tar.gz
    [packet-images]$ git add Dockerfile .gitattributes image.tar.gz
    [packet-images]$ git commit -m "Add Mellanox package for supermachine1"
    [packet-images]$ git push origin ubuntu_17_10-supermachine1
    # get latest commit sha (latest image tag)
    [packet-images]$ git rev-parse --verify HEAD

Using build script method:

    [packet-images]$ ./tools/build.sh -d debian_9 -t baremetal_0 -a x86_64 -b debian_9-baremetal_0-dev
    Checking out debian_9-baremetal_0-dev...
    Switched to and reset branch 'debian_9-baremetal_0-dev'
    Create read-tree for debian_9-base...
    Create read-tree for debian_9-baremetal_0...
    Build debian_9-base with docker...
    Build debian_9-baremetal_0 with docker...
    Save image

### Deploying your custom image on Packet
You have a working image built, so now what?
Run it using our custom_image feature, or use it via iPXE/Custom OS.
