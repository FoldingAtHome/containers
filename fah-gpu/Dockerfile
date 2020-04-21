# Folding@home GPU Container

# CUDA 9.2 is chosen to be compatable with all drivers v396+
FROM nvidia/cuda:9.2-base-ubuntu18.04
LABEL description="Official Folding@home GPU Container"

RUN groupadd -g 9999 fahuser && \
    useradd -r -d /fah -u 9999 -g fahuser fahuser 

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      ocl-icd-opencl-dev \
      clinfo \
      curl \
    # point at lib mapped in by container runtime
    && mkdir -p /etc/OpenCL/vendors \
    && echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd \
    # next line gets past the fahclient.postinst
    && mkdir -p /etc/fahclient && touch /etc/fahclient/config.xml \
    # download and verify checksum
    && curl -fsSL \
      https://download.foldingathome.org/releases/public/release/fahclient/debian-stable-64bit/v7.5/fahclient_7.5.1_amd64.deb \
      -o fah.deb \
    && echo "efe47061d328667259c1815b98bc6c37da339363fb8e9386673f6a96f4d37d26 fah.deb" \
      | sha256sum -c --strict - \
    # install and cleanup
    && DEBIAN_FRONTEND=noninteractive dpkg --install --force-depends fah.deb \
    && rm -rf fah.deb \
    && apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* \
    && mkdir -p /fah \
    && chown fahuser:fahuser /fah \
    && chmod 750 /fah

COPY README.md /

WORKDIR "/fah"
EXPOSE 7396 36330

USER fahuser

ENTRYPOINT ["/usr/bin/FAHClient", "--chdir", "/fah"]
