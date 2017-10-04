# Copyright 2017 Reza Farrahi M
# Copyright 2016 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# This file creates a standard build environment for building cross
# platform go binary for the architecture kubernetes cares about.

FROM imriss/archlinux
RUN echo 'K8s cross on Arch Linux'
MAINTAINER Reza Farrahi <imriss@ieee.org>
LABEL description="K8s cross / Arch Linux"

RUN  pacman -Syyu --noconfirm && \
  pacman -S findutils nano vi --noconfirm --needed && \
  pacman-db-upgrade && \
  export editor=nano && \
  pacman -S --noconfirm --needed systemd python python-yaml wget python-pip openssh git curl jq yajl perl expac \
  gcc gcc-libs clang && \
  pip install --upgrade pip && \
  pip install simplejson

ADD ./aur.sh /usr/sbin/aur.sh
ADD ./add-aur.sh /usr/sbin/add-aur
RUN chmod u+x /usr/sbin/aur.sh
RUN chmod u+x /usr/sbin/add-aur
RUN export "PATH=/usr/bin/core_perl:$PATH" && \
  add-aur docker

RUN su docker -c 'pacaur -S --noprogressbar --noedit --noconfirm --needed base-devel \
  expect cmake gcc-fortran protobuf python-protobuf '

ENV GOARM 7
ENV KUBE_DYNAMIC_CROSSPLATFORMS \
  armhf \
  arm64 \
  s390x \
  ppc64el

ENV KUBE_CROSSPLATFORMS \
  linux/386 \
  linux/arm linux/arm64 \
  linux/ppc64le \
  linux/s390x \
  darwin/amd64 darwin/386 \
  windows/amd64 windows/386

# Pre-compile the standard go library when cross-compiling. This is much easier now when we have go1.5+
RUN for platform in ${KUBE_CROSSPLATFORMS}; do GOOS=${platform%/*} GOARCH=${platform##*/} go install cgo std; done

# Use dynamic cgo linking for architectures other than amd64 for the server platforms
# To install crossbuild essential for other architectures add the following repository.
RUN for platform in ${KUBE_DYNAMIC_CROSSPLATFORMS}; do apt-get install -y crossbuild-essential-${platform}; done \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# work around 64MB tmpfs size in Docker 1.6
ENV TMPDIR /tmp.k8s
RUN mkdir $TMPDIR \
  && chmod a+rwx $TMPDIR \
  && chmod o+t $TMPDIR

# Get the code coverage tool and goimports
RUN go get golang.org/x/tools/cmd/cover \
            golang.org/x/tools/cmd/goimports

# Download and symlink etcd. We need this for our integration tests.
RUN export ETCD_VERSION=v3.1.10; \
  mkdir -p /usr/local/src/etcd \
  && cd /usr/local/src/etcd \
  && curl -fsSL https://github.com/coreos/etcd/releases/download/${ETCD_VERSION}/etcd-${ETCD_VERSION}-linux-amd64.tar.gz | tar -xz \
&& ln -s ../src/etcd/etcd-${ETCD_VERSION}-linux-amd64/etcd /usr/local/bin/
