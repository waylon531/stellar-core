#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------
FROM ubuntu:focal

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN=true

# This Dockerfile adds a non-root 'vscode' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git iproute2 procps lsb-release \
    #
    # Try to suppress any future interactive tzdata installation nonsense
    && ln -fs /usr/share/zoneinfo/UTC /etc/localtime \
    && apt-get -y install tzdata \
    && dpkg-reconfigure --frontend noninteractive tzdata \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root users
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Work around https://github.com/sudo-project/sudo/issues/42
    && echo "Set disable_coredump false" >> /etc/sudo.conf \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Add test tool chain
# NOTE: newer version of the compilers are not
#    provided by stock distributions
#    and are provided by the /test toolchain
RUN sudo apt-get update \
    && sudo apt-get -y install software-properties-common \
    && sudo add-apt-repository ppa:ubuntu-toolchain-r/test \
    && sudo apt-get update

# Install common compilation tools
RUN sudo apt-get -y install git build-essential pkg-config autoconf automake libtool bison flex libpq-dev parallel bear

# Update compiler tools
RUN sudo apt-get -y install clang clangd clang-tools libc++-dev libc++abi-dev

# Install postgresql to enable tests under make check
RUN sudo apt-get -y install postgresql

# Set up locale
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && locale-gen

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Install clangd for code navigation and clang-format for formatting, and link them into a directory in the $PATH
# (For now, our standard is an old version of clang-format, 5.0.2)
RUN sudo apt-get -y install curl
RUN curl https://releases.llvm.org/5.0.2/clang+llvm-5.0.2-x86_64-linux-gnu-ubuntu-16.04.tar.xz | tar -xJf - -C /usr/local
RUN ln -sf /usr/local/clang+llvm-5.0.2-x86_64-linux-gnu-ubuntu-16.04/bin/clang-format /usr/bin

# Switch to non-root user
ENV HOME=/home/vscode
WORKDIR $HOME
USER vscode

# Use clang for its formatting and code-navigation
ENV CC=clang
ENV CXX=clang++
ENV CFLAGS="-O3 -g1 -fno-omit-frame-pointer"
ENV CXXFLAGS="$CFLAGS -stdlib=libc++ -I/usr/include/libcxxabi"

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=
ENV DEBCONF_NONINTERACTIVE_SEEN=
