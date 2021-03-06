# Build Julia from source on your x86-64 machine, 
# which means it is not instruction for cross compilation, 
# but it will helpful for one who is trying to create script
# to build Julia from source for any device.

FROM ubuntu:18.04

MAINTAINER SATOSHI TERASAKI

RUN apt-get update && apt-get install -y \
    build-essential \
    libatomic1 \
    python \
    gfortran \
    perl \
    wget \
    m4 \
    cmake \
    pkg-config \
    git \
    && apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/*

ARG JL_VERSION="v1.5.4"
ARG WDIR="/root"
ARG JL_BUILD_DIR=$WDIR/build
WORKDIR $JL_BUILD_DIR

RUN echo "\
CXXFLAGS=-D_GLIBCXX_USE_CXX11_ABI=0\n\
prefix=/usr/local/julia-$JL_VERSION\n\
USE_BINARYBUILDER = 0\n\
" > Make.user \
    && cat Make.user \
    && git clone --depth=1 -b $JL_VERSION https://github.com/JuliaLang/julia.git julia-$JL_VERSION\
    && cp Make.user $JL_BUILD_DIR/julia-$JL_VERSION \
    && cd julia-$JL_VERSION \
    && make -j $(nproc) \
    && make install \
    && echo "clean up $JL_BUILD_DIR" \
    && rm -r $JL_BUILD_DIR \
    && echo "Done"

ENV PATH=/usr/local/julia-$JL_VERSION/bin:$PATH
# runtime test
RUN julia -e "using InteractiveUtils; versioninfo()"

CMD ["julia"]
