FROM ubuntu:18.04 AS toolchain
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      build-essential libncurses5-dev python unzip \
      git gawk wget file curl sudo runit pkg-config
RUN useradd -m build
RUN echo 'build ALL=(ALL)	NOPASSWD: ALL' >>/etc/sudoers
USER build
WORKDIR /home/build
RUN mkdir share openwrt openwrt/staging_dir
COPY toolchain-arm_cortex-a9+vfpv3_gcc-4.8-linaro_musl-1.1.15_eabi.tar.gz .
RUN tar -C openwrt/staging_dir -f toolchain-arm_cortex-a9+vfpv3_gcc-4.8-linaro_musl-1.1.15_eabi.tar.gz -x
RUN rm toolchain-arm_cortex-a9+vfpv3_gcc-4.8-linaro_musl-1.1.15_eabi.tar.gz
ENV STAGING_DIR=/home/build/openwrt/staging_dir
ARG TOOLCHAIN_PATH=${STAGING_DIR}/toolchain-arm_cortex-a9+vfpv3_gcc-4.8-linaro_musl-1.1.15_eabi
ENV TOOLCHAIN_PATH=${TOOLCHAIN_PATH}
ENV CC=${TOOLCHAIN_PATH}/bin/arm-openwrt-linux-gcc
ENV CXX=${TOOLCHAIN_PATH}/bin/arm-openwrt-linux-g++
ENV CROSS_COMPILE=arm-openwrt-linux-
ENV SYSROOT=${TOOLCHAIN_PATH}
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
