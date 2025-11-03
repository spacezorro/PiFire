# Dockerfile is for github workflows to run the install script against

# 32 or 64
ARG TARGETVARIANT

# workflow builds trixie or you could try sid?
ARG OS=bookworm

# Build args to pass to install.sh
ARG INSTALL_FLAG=""

FROM --platform=linux/arm64 debian:${OS} AS base64
FROM --platform=linux/arm/v7 debian:${OS} AS base32

FROM base${TARGETVARIANT} AS base

#ENV UDEV=off
ENV LC_ALL=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get upgrade -y

# Install stuff to make debian:whatever match raspbian:whatever
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get install -y --no-install-recommends \
    apt-utils curl vim wget htop net-tools git ca-certificates gnupg
# Install the keyring
RUN CODENAME=$(lsb_release -sc) && \
    mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://archive.raspberrypi.org/debian/raspberrypi.gpg.key | \
    gpg --dearmor -o /etc/apt/keyrings/raspberrypi-archive-keyring.gpg && \
    chmod 644 /etc/apt/keyrings/raspberrypi-archive-keyring.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/raspberrypi-archive-keyring.gpg] http://archive.raspberrypi.org/debian/ $CODENAME main" > /etc/apt/sources.list.d/raspi.list && \
    apt update
# Install "raspbian" without the kernel
RUN cd /tmp \ 
    && git clone https://github.com/RPi-Distro/pi-gen.git \
    && cd pi-gen \
    && case "$(sh -c '. /etc/os-release; echo $VERSION_CODENAME')-$(dpkg --print-architecture)" in \
        bookworm-arm64) git checkout bookworm-arm64 ;; \
        bookworm-armhf) git checkout bookworm ;; \
        bullseye-arm64) git checkout bullseye-arm64 ;; \
        bullseye-armhf) git checkout bullseye ;; \
        *-arm64) git checkout arm64 ;; \
        *-armhf) git checkout master ;; \
    esac \
    && cat $(find stage0 stage1 stage2 -name '*-packages' -type f) | sed 's/ /\n/g' | sort | grep -v "linux-image" > /tmp/packages.txt \
    && cat $(find stage0 stage1 stage2 -name '*-packages-nr' -type f) | sed 's/ /\n/g' | sort > /tmp/packages-nr.txt \
    && cd /tmp \
    && rm -rf /tmp/pi-gen \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 82B129927FA3303E \
    && apt-get update --allow-releaseinfo-change \
    && apt-get install -fy --no-install-recommends $(cat /tmp/packages-nr.txt) \
    && apt-get install -fy $(cat /tmp/packages.txt) \
    && rm -rf /tmp/packages-nr.txt /tmp/packages.txt

# Copy install script
COPY install.sh /install.sh
RUN chmod +x /install.sh

# Run install.sh with the correct flag
ENV TESTINSTALL=true

CMD ["/bin/bash"]
