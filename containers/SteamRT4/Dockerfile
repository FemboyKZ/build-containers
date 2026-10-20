FROM registry.gitlab.steamos.cloud/steamrt/steamrt4/sdk:latest-container-runtime-depot AS sdk

ENV DEBIAN_FRONTEND=noninteractive

# Install Clang 22, LLD and libssl-dev:i386 (not part of this SDK)
RUN wget -4 --retry-on-host-error -O /etc/apt/keyrings/apt.llvm.org.asc https://apt.llvm.org/llvm-snapshot.gpg.key
RUN echo "deb [signed-by=/etc/apt/keyrings/apt.llvm.org.asc] http://apt.llvm.org/trixie/ llvm-toolchain-trixie-22 main" >> /etc/apt/sources.list.d/llvm.list
RUN echo "deb-src [signed-by=/etc/apt/keyrings/apt.llvm.org.asc] http://apt.llvm.org/trixie/ llvm-toolchain-trixie-22 main" >> /etc/apt/sources.list.d/llvm.list
RUN apt update -y && apt install --no-install-recommends --no-install-suggests -y clang-22 lld-22 libssl-dev:i386

# Install AMBuild
RUN git clone https://github.com/alliedmodders/ambuild
RUN cd ambuild && python setup.py install
RUN rm -rf ambuild

# Install Premake5
# RUN wget --retry-on-host-error -O premake.tar.gz https://github.com/premake/premake-core/archive/refs/tags/v5.0.0-beta8.tar.gz
# RUN tar -xzf premake.tar.gz
# RUN make -C premake-core-5.0.0-beta8 -f Bootstrap.mak linux
# RUN install premake-core-5.0.0-beta8/bin/release/premake5 /usr/local/bin/premake5
# RUN rm -rf premake*

# Install xmake
# RUN curl -fsSL https://xmake.io/shget.text | bash -s v3.1.0
# RUN ln -sf /root/.local/bin/xmake /usr/bin/xmake
# ENV XMAKE_ROOT=y

# Cleanup
# Builds use about a tenth of the SDK, so everything not reachable from KEEP is purged.
# A build that needs another system package has to list it here.
# The x86 entries mirror what clang -m32 selects in the full SDK. The lib32 multilib packages build different binaries.
# python-is-python3 is the ambuild shebang, sudo refuses to be removed.
ARG KEEP="clang-22 lld-22 libstdc++-14-dev libc6-dev binutils \
    libstdc++-14-dev:i386 libstdc++-14-dev-i386-cross libc6-dev:i386 \
    libcurl4-openssl-dev libssl-dev libssl-dev:i386 \
    git python3 python-is-python3 python3-setuptools \
    ca-certificates curl wget make zip unzip xz-utils sudo"
RUN apt-mark auto $(dpkg-query -W -f='${binary:Package}\n') > /dev/null \
    && apt-mark manual $KEEP \
    && apt autoremove -y --purge \
    -o APT::AutoRemove::RecommendsImportant=false \
    -o APT::AutoRemove::SuggestsImportant=false \
    && apt clean -y \
    && rm -rf /var/lib/apt/lists/* /var/cache/* /var/log/* /usr/share/doc /usr/share/man /usr/share/locale

# The purged clang package owns /usr/bin/clang, so the links go in after it
RUN ln -sf /usr/bin/clang-22 /usr/bin/clang && ln -sf /usr/bin/clang++-22 /usr/bin/clang++ && ln -sf /usr/bin/ld.lld-22 /usr/bin/ld.lld

# Deleting in a layer on top of the SDK would not shrink the pull, so the result is copied into a fresh image
FROM scratch
LABEL author="FemboyKZ"
LABEL org.opencontainers.image.description="Docker container used as a build environment for our plugins"
LABEL org.opencontainers.image.source="https://github.com/FemboyKZ/build-containers"

COPY --from=sdk / /

ENV DEBIAN_FRONTEND=noninteractive
CMD ["/bin/bash"]
