FROM alpine:3.15
# In case the main package repositories are down, use the alternative base image:
# FROM gliderlabs/alpine:3.4
# Updated by nbp - adding ca-certs etc.

MAINTAINER Nikyle Nguyen <NLKNguyen@MSN.com>

ARG REQUIRE="sudo build-base"
RUN apk update && apk upgrade \
      && apk add --no-cache ${REQUIRE}
RUN apk update && apk add ca-certificates && update-ca-certificates && apk add openssl \
  curl-dev \
  curl \
  zlib-dev \
  bash \
  cmake \
  m4 \
  libexecinfo-dev \
  libexecinfo \
  linux-headers

RUN apk add \
  --no-cache \
  --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing \
  --repository http://dl-cdn.alpinelinux.org/alpine/edge/main \
  hdf5-dev \
  hdf5 \
  netcdf-dev \
  netcdf \
  openmp-dev \
  openmp

#### INSTALL MPICH ####
# Source is available at http://www.mpich.org/static/downloads/

# Build Options:
# See installation guide of target MPICH version
# Ex: http://www.mpich.org/static/downloads/3.2/mpich-3.2-installguide.pdf
# These options are passed to the steps below
ARG MPICH_VERSION="3.2"
ARG MPICH_CONFIGURE_OPTIONS="--disable-fortran"
ARG MPICH_MAKE_OPTIONS

# Download, build, and install MPICH
RUN mkdir /tmp/mpich-src
WORKDIR /tmp/mpich-src
RUN wget http://www.mpich.org/static/downloads/${MPICH_VERSION}/mpich-${MPICH_VERSION}.tar.gz \
      && tar xfz mpich-${MPICH_VERSION}.tar.gz  \
      && cd mpich-${MPICH_VERSION}  \
      && ./configure ${MPICH_CONFIGURE_OPTIONS}  \
      && make ${MPICH_MAKE_OPTIONS} && make install \
      && rm -rf /tmp/mpich-src


#### TEST MPICH INSTALLATION ####
RUN mkdir /tmp/mpich-test
WORKDIR /tmp/mpich-test
COPY mpich-test .
RUN sh test.sh
RUN rm -rf /tmp/mpich-test

#### Download, build, and install Repast HPC
RUN mkdir /tmp/repast-hpc-src/
WORKDIR /tmp/repast-hpc-src
RUN wget https://github.com/clinejc/repast.hpc/archive/refs/heads/netcdf-cxx4-install.zip -O repast.hpc-netcdf-cxx4-install.zip \
  && unzip repast.hpc-netcdf-cxx4-install.zip

#### Install NetCDF
WORKDIR /tmp/repast-hpc-src/repast.hpc-netcdf-cxx4-install/dist/Manual/with-netcdf4
RUN /bin/bash install.sh netcdf_cxx4

#### Install boost
WORKDIR /tmp/repast-hpc-src/repast.hpc-netcdf-cxx4-install/dist/Manual/with-netcdf4
RUN /bin/bash install.sh boost

#### Install googletest
WORKDIR /tmp/repast-hpc-src/repast.hpc-netcdf-cxx4-install/dist/Manual/with-netcdf4
RUN /bin/bash install.sh googletest

#### Install repast_hpc
WORKDIR /tmp/repast-hpc-src/repast.hpc-netcdf-cxx4-install/dist/Manual/with-netcdf4
RUN /bin/bash install.sh repast_hpc

#### CREATE WORKING DIRECTORY FOR USER ####
ARG WORKDIR=/project
ENV WORKDIR ${WORKDIR}
RUN mkdir ${WORKDIR}
RUN mkdir ${WORKDIR}/examples

#### Move repast_hpc examples to working directory ####
RUN mv /tmp/repast-hpc-src/repast.hpc-netcdf-cxx4-install/build/src/rumor_model/bin/rumor ${WORKDIR}/examples
RUN mv /tmp/repast-hpc-src/repast.hpc-netcdf-cxx4-install/build/src/zombies/bin/zombies ${WORKDIR}/examples

#### CLEAN UP Repast HPC ####
WORKDIR /
RUN rm -rf /tmp/tmp/repast-hpc-src


#### CLEAN UP ####
#WORKDIR /
#RUN rm -rf /tmp/*


#### ADD DEFAULT USER ####
ARG USER=mpi
ENV USER ${USER}
RUN adduser -D ${USER} \
      && echo "${USER}   ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

ENV USER_HOME /home/${USER}
RUN chown -R ${USER}:${USER} ${USER_HOME}

#### CREATE WORKING DIRECTORY FOR USER ####
ARG WORKDIR=/project
ENV WORKDIR ${WORKDIR}
#RUN mkdir ${WORKDIR}
RUN chown -R ${USER}:${USER} ${WORKDIR}

WORKDIR ${WORKDIR}
USER ${USER}

USER root

CMD ["/bin/ash"]
