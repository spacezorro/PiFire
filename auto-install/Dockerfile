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

# Copy install script
COPY install.sh /install.sh
RUN chmod +x /install.sh

# Run install.sh with the correct flag
ENV TESTINSTALL=true
RUN /install.sh ${INSTALL_FLAG}

CMD ["/bin/bash"]
