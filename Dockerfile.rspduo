FROM debian:bookworm as blah2_rspduo
LABEL maintainer="Jehan <jehan.azad@gmail.com>"
LABEL org.opencontainers.image.source https://github.com/30hours/blah2

WORKDIR /opt/blah2
ADD lib lib

# Install only essential dependencies for RSPduo
RUN apt-get update && apt-get install -y \
    g++ \
    make \
    cmake \
    ninja-build \
    git \
    curl \
    zip \
    unzip \
    libfftw3-dev \
    pkg-config \
    gfortran \
    libboost-all-dev \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Install only required vcpkg dependencies (skip unused ones)
ENV VCPKG_ROOT=/opt/vcpkg
ENV CC=/usr/bin/gcc
ENV CXX=/usr/bin/g++

# Create minimal vcpkg.json for RSPduo only
COPY <<EOF /opt/blah2/lib/vcpkg.json
{
  "dependencies": [
    "rapidjson",
    "asio",
    "ryml",
    "httplib",
    "armadillo",
    "catch2"
  ]
}
EOF

RUN export PATH="/opt/vcpkg:${PATH}" \
    && git clone --depth 1 https://github.com/microsoft/vcpkg /opt/vcpkg \
    && if [ "$(uname -m)" = "aarch64" ]; then export VCPKG_FORCE_SYSTEM_BINARIES=1; fi \
    && /opt/vcpkg/bootstrap-vcpkg.sh -disableMetrics \
    && cd /opt/blah2/lib && vcpkg install --clean-after-build

# Install SDRplay API (keep existing logic)
USER root
RUN export ARCH=$(uname -m) \
    && if [ "$ARCH" = "x86_64" ]; then \
         ARCH="amd64"; \
    elif [ "$ARCH" = "aarch64" ]; then \
         ARCH="arm64"; \
    fi \
    && export MAJVER="3.15" \
    && export MINVER="" \
    && export VER=${MAJVER}.${MINVER} \
    && cd /opt/blah2/lib/sdrplay-${VER} \
    && chmod +x SDRplay_RSP_API-Linux-${VER}.run \
    && ./SDRplay_RSP_API-Linux-${VER}.run --noexec --target /opt/blah2/lib/sdrplay-${VER}/extract \
    && cp -r /opt/blah2/lib/sdrplay-${VER}/extract/* /opt/blah2/lib/sdrplay-${VER}/ \
    && cp ${ARCH}/libsdrplay_api.so.${MAJVER} /usr/local/lib/libsdrplay_api.so.${MAJVER} \
    && cp inc/* /usr/local/include \
    && chmod 644 /usr/local/lib/libsdrplay_api.so.${MAJVER}

FROM blah2_rspduo as blah2
LABEL maintainer="Jehan <jehan.azad@gmail.com>"

WORKDIR /opt/blah2

ADD src src
ADD test test
ADD CMakeLists.txt CMakePresets.json Doxyfile ./

# Optimized build with parallel jobs
RUN set -ex \
    && mkdir -p build \
    && cd build \
    && cmake -S .. --preset prod-release \
        -DCMAKE_PREFIX_PATH=$(echo /opt/blah2/lib/vcpkg_installed/*/share) .. \
    && cd prod-release \
    && make -j$(nproc) \
    && echo "==== Binary location: ====" \
    && ls -l /opt/blah2/bin/blah2 \
    && mkdir -p /blah2/bin \
    && cp -v /opt/blah2/bin/blah2 /blah2/bin/ \
    && chmod +x /blah2/bin/blah2

WORKDIR /blah2/bin
