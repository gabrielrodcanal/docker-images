# STAGE 1: Clone
FROM ubuntu:20.04 AS cloner

RUN apt-get update && \
    apt-get -y install wget git
RUN git clone --depth 1 https://github.com/Xilinx/hls-llvm-project.git
WORKDIR hls-llvm-project
RUN git fetch origin tag 2023.2 && \
    git checkout tags/2023.2

# STAGE 2: Build LLVM
FROM ubuntu:20.04 AS llvm-builder
COPY --from=cloner /hls-llvm-project /hls-llvm-project
RUN apt-get update && \
    apt-get install -y make lld clang ccache wget
RUN wget https://github.com/Kitware/CMake/releases/download/v3.31.5/cmake-3.31.5-linux-x86_64.sh && \
    bash cmake-3.31.5-linux-x86_64.sh --skip-license --prefix=/opt && \
    ln -s /opt/bin/cmake /usr/local/bin/cmake
WORKDIR hls-llvm-project
RUN bash build-hls-llvm-project.sh

# STAGE 3: Packing
FROM ubuntu:20.04
COPY --from=llvm-builder /hls-llvm-project/hls-build/bin/opt /opt/hls-build/bin/opt
COPY --from=llvm-builder /hls-llvm-project/hls-build/bin/llvm-dis /opt/hls-build/bin/llvm-dis
COPY --from=llvm-builder /hls-llvm-project/hls-build/bin/llvm-as /opt/hls-build/bin/llvm-as
COPY --from=llvm-builder /hls-llvm-project/hls-build/bin/clang /opt/hls-build/bin/clang
COPY --from=llvm-builder /hls-llvm-project/ext /opt/ext
RUN echo "export PATH=/opt/hls-build/bin:$PATH" >> ~/.bashrc
RUN echo "export LD_LIBRARY_PATH=/opt/ext/sqlite-3.28.0/lib/lnx64/:$LD_LIBRARY_PATH" >> ~/.bashrc
WORKDIR /workdir
