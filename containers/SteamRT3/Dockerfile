FROM registry.gitlab.steamos.cloud/steamrt/sniper/sdk:latest-container-runtime-depot
LABEL author="FemboyKZ"
LABEL org.opencontainers.image.description="Docker container used as a build environment for our plugins"
LABEL org.opencontainers.image.source="https://github.com/FemboyKZ/build-containers"

ENV DEBIAN_FRONTEND=noninteractive

# Remove expired release info to avoid apt update errors
RUN printf '%s\n' \
    'Acquire::Check-Valid-Until "false";' \
    'Acquire::AllowReleaseInfoChange::Suite "true";' \
    'Acquire::AllowReleaseInfoChange::Version "true";' \
    > /etc/apt/apt.conf.d/99-ci-ignore-expired-release

RUN for list in /etc/apt/sources.list /etc/apt/sources.list.d/*.list; do \
    [ -f "$list" ] || continue; \
    sed -i -E 's@^[[:space:]]*deb(-src)?[[:space:]].*deb\.debian\.org/debian-security.*@# & (disabled: bullseye EOL)@' "$list"; \
done

# Update
RUN apt update -y && apt upgrade -y

# Install Clang 22
RUN echo "deb http://apt.llvm.org/bullseye/ llvm-toolchain-bullseye-22 main" >> /etc/apt/sources.list.d/llvm.list
RUN echo "deb-src http://apt.llvm.org/bullseye/ llvm-toolchain-bullseye-22 main" >> /etc/apt/sources.list.d/llvm.list
RUN wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -
RUN apt update -y && apt install --no-install-recommends --no-install-suggests -y clang-22
RUN ln -sf /usr/bin/clang-22 /usr/bin/clang && ln -sf /usr/bin/clang++-22 /usr/bin/clang++

# Install AMBuild
RUN git clone https://github.com/alliedmodders/ambuild
RUN cd ambuild && python setup.py install
RUN rm -rf ambuild

# Install Premake5
RUN wget -O premake.tar.gz https://github.com/premake/premake-core/archive/refs/tags/v5.0.0-beta8.tar.gz
RUN tar -xzf premake.tar.gz
RUN make -C premake-core-5.0.0-beta8 -f Bootstrap.mak linux
RUN install premake-core-5.0.0-beta8/bin/release/premake5 /usr/local/bin/premake5
RUN rm -rf premake*

# Install xmake
RUN curl -fsSL https://xmake.io/shget.text | bash -s v3.1.0
RUN ln -sf /root/.local/bin/xmake /usr/bin/xmake
ENV XMAKE_ROOT=y

# Cleanup
RUN apt clean -y && apt autoremove -y && rm -rf /var/lib/apt/lists/*
