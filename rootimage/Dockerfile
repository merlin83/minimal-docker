# adapted from https://github.com/radial/core-busyboxplus/blob/master/curl%2Ftarmaker%2FDockerfile
FROM debian:jessie

MAINTAINER merlin83

ADD files/fs/etc/apt/ /etc/apt/

# install minimum required packages
RUN apt-get update && DEBIAN_FRONTEND="noninteractive" APT_LISTCHANGES_FRONTEND="none" \
    apt-get -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" \
      -y --force-yes -q --no-install-recommends install \
            wget \
            build-essential \
            libncurses-dev \
            rsync \
            unzip \
            bc \
            gnupg \
            cpio \
            python

RUN mkdir -p /tmp/buildroot/
WORKDIR /tmp/buildroot

# Retrieve files and check authenticity
ENV BR_VERSION 2014.08
RUN wget -nv http://buildroot.uclibc.org/downloads/buildroot-$BR_VERSION.tar.gz &&\
    wget -nv http://buildroot.uclibc.org/downloads/buildroot-$BR_VERSION.tar.gz.sign &&\
    wget -nv http://uclibc.org/~jacmet/pubkey.gpg &&\
    gpg --import pubkey.gpg &&\
    gpg --verify buildroot-$BR_VERSION.tar.gz.sign

ENV TEST 123

# Extract
RUN tar -zxf buildroot-$BR_VERSION.tar.gz &&\
    mv -f buildroot-$BR_VERSION buildroot

# Add necessary files (packages) into files/fs/tmp/buildroot/
ADD files/fs/tmp/buildroot/ /tmp/buildroot/

WORKDIR /tmp/buildroot/buildroot

# Configure base buildroot image
RUN make defconfig

# Add base config
RUN sed -i 's/BR2_i386=y/BR2_x86_64=y/' .config &&\
    echo "BR2_TOOLCHAIN_BUILDROOT_LARGEFILE=y\n" >> .config &&\
    echo "BR2_TOOLCHAIN_BUILDROOT_INET_IPV6=y\n" >> .config &&\
    echo "BR2_TOOLCHAIN_BUILDROOT_WCHAR=y\n" >> .config

# Add image flavor
RUN echo "BR2_PACKAGE_LIBCURL=y\n" >> .config &&\
    echo "BR2_PACKAGE_CURL=y\n" >> .config &&\
    echo "BR2_PACKAGE_BUSYBOX=y\n" >> .config &&\
    echo "BR2_PACKAGE_DROPBEAR=y\n" >> .config &&\
    echo "BR2_PACKAGE_RSYNC=y\n" >> .config &&\
#    echo "BR2_PACKAGE_GIT=y\n" >> .config &&\
#    echo "BR2_PACKAGE_OPKG=y\n" >> .config &&\
    echo "BR2_PACKAGE_OPENSSL=y\n" >> .config &&\
    echo "BR2_PACKAGE_LIBSSH2=y\n" >> .config &&\
    echo "BR2_PACKAGE_CA_CERTIFICATES=y\n" >> .config

# Reformat config and prep for make
RUN make oldconfig

# Make
RUN make --quiet

# Done!
RUN cp /tmp/buildroot/buildroot/output/images/rootfs.tar /tmp/rootfs.tar
WORKDIR /tmp
RUN md5sum rootfs.tar > rootfs.tar.md5
