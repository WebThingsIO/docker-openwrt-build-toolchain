# This follows the multi-stage builds pattern described here:
# https://docs.docker.com/develop/develop-images/multistage-build/
#
# By doing this, the final toolchain image is about 5 Gb smaller than the
# toolchain-builder image.
FROM ubuntu:18.04 AS toolchain-base
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      build-essential libncurses5-dev python unzip \
      git gawk wget file curl sudo runit pkg-config
RUN useradd -m build
RUN echo 'build ALL=(ALL)	NOPASSWD: ALL' >>/etc/sudoers
USER build
WORKDIR /home/build
RUN mkdir share

FROM toolchain-base as toolchain-builder
USER build
WORKDIR /home/build
RUN git clone https://github.com/openwrt/openwrt
WORKDIR /home/build/openwrt
# This was master when we first built 0.8.1
ARG VERSION=9424b6f998917e2926c0b7afb8d7a968590da335
RUN git checkout $VERSION
RUN make package/symlinks
COPY config.toolchain .
RUN cp config.toolchain .config && make defconfig
RUN make -j4 toolchain/install
ENTRYPOINT bash

FROM toolchain-base as toolchain
USER build
WORKDIR /home/build
ENV STAGING_DIR=/home/build/openwrt/staging_dir
ARG TOOLCHAIN_PATH=${STAGING_DIR}/toolchain-arm_cortex-a7+neon-vfpv4_gcc-7.4.0_musl_eabi
ENV TOOLCHAIN_PATH=${TOOLCHAIN_PATH}
ENV CC=${TOOLCHAIN_PATH}/bin/arm-openwrt-linux-gcc
ENV CXX=${TOOLCHAIN_PATH}/bin/arm-openwrt-linux-g++
ENV CROSS_COMPILE=arm-openwrt-linux-
ENV SYSROOT=${TOOLCHAIN_PATH}
COPY --from=toolchain-builder ${TOOLCHAIN_PATH}/ ${TOOLCHAIN_PATH}/
RUN sudo bash -c "echo 'owrt-user ALL=(ALL)	NOPASSWD: ALL' >>/etc/sudoers"
RUN sudo mkdir /owrt
COPY owrt /owrt
COPY entrypoint.sh /owrt
RUN sudo chmod +x /owrt/entrypoint.sh
# Create the .owrt file which is used by build-adapter.sh script from the
# the addon-builder. We need to put this into a file since the ENV statements
# above get "lost" when entrypoint.sh runs chpst.
RUN sh -c 'echo export STAGING_DIR='${STAGING_DIR}' > .owrt'
RUN sh -c 'echo export TOOLCHAIN_PATH='${TOOLCHAIN_PATH}' >> .owrt'
RUN sh -c 'echo export CC='${CC}' >> .owrt'
RUN sh -c 'echo export CXX='${CXX}' >> .owrt'
RUN sh -c 'echo export CROSS_COMPILE='${CROSS_COMPILE}' >> .owrt'
RUN sh -c 'echo export SYSROOT='${SYSROOT}' >> .owrt'
RUN sh -c 'echo export PATH=\"'${TOOLCHAIN_PATH}'/bin:\${PATH}\" >> .owrt'
WORKDIR /build
ENTRYPOINT [ "/owrt/entrypoint.sh" ]
