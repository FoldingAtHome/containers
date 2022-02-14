# Folding@home AMD ROCm Container

FROM ubuntu:18.04
LABEL description="Folding@home AMD ROCm Container"

ARG ROCM_RELEASE=5.0
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      kmod \
      python3 \
      curl \
      gnupg \
      ca-certificates \
      libnuma-dev \
    # get ROCm OpenCL runtime packages
    && curl -sL https://repo.radeon.com/rocm/rocm.gpg.key | apt-key add - \
    && printf "deb [arch=amd64] https://repo.radeon.com/rocm/apt/${ROCM_RELEASE}/ ubuntu main" \
        | tee /etc/apt/sources.list.d/rocm.list \
    && apt-get update && apt-get install -y --no-install-recommends \
      rocm-opencl-runtime \
      rocm-smi-lib \
      rocminfo \
    # next line gets past the fahclient.postinst
    && mkdir -p /etc/fahclient && touch /etc/fahclient/config.xml \
    # download and verify checksum
    && curl -fsSL \
      https://download.foldingathome.org/releases/public/release/fahclient/debian-stable-64bit/v7.6/fahclient_7.6.21_amd64.deb \
      -o fah.deb \
    && echo "2827f05f1c311ee6c7eca294e4ffb856c81957e8f5bfc3113a0ed27bb463b094 fah.deb" \
      | sha256sum -c --strict - \
    # install and cleanup
    && dpkg --install --force-depends fah.deb \
    && rm -rf fah.deb \
    && apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# explicitly point FAHClient to the ROCm libOpenCL
#ENV LD_LIBRARY_PATH=/opt/rocm-${ROCM_VER}/lib
#ENV PATH=$PATH:/opt/rocm-${ROCM_VER}/bin:/opt/rocm-${ROCM_VER}/opencl/bin
ENV LD_LIBRARY_PATH=/opt/rocm/lib
ENV PATH=$PATH:/opt/rocm/bin:/opt/rocm/opencl/bin

WORKDIR "/fah"
VOLUME ["/fah"]
EXPOSE 7396 36330

ENTRYPOINT ["/usr/bin/FAHClient", "--chdir", "/fah"]
