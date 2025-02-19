# STAGE 1: Clone
FROM ubuntu:20.04 as cloner

RUN apt-get update && \
    apt-get -y install wget git
RUN git clone --depth 1 https://github.com/openxla/stablehlo.git 

WORKDIR /stablehlo
RUN git fetch origin tag v1.9.2 && \
    git checkout tags/v1.9.2 && \
    git clone --depth 1 https://github.com/llvm/llvm-project.git
WORKDIR /stablehlo/llvm-project
RUN git fetch origin $(cat ../build_tools/llvm_version.txt) && \ 
    git checkout $(cat ../build_tools/llvm_version.txt)

# STAGE 2: Build LLVM
FROM ubuntu:20.04 as llvm-builder
COPY --from=cloner /stablehlo /stablehlo
WORKDIR /stablehlo
RUN apt-get update && \
    apt-get -y install ninja-build lld clang ccache wget
RUN wget https://github.com/Kitware/CMake/releases/download/v3.31.5/cmake-3.31.5-linux-x86_64.sh && \
    bash cmake-3.31.5-linux-x86_64.sh --skip-license --prefix=/opt && \
    ln -s /opt/bin/cmake /usr/local/bin/cmake 
RUN cmake --version
RUN CC=clang CXX=clang++ MLIR_ENABLE_BINDINGS_PYTHON=OFF build_tools/build_mlir.sh $(pwd)/llvm-project/ $(pwd)/llvm-build

# STAGE 3: Build StableHLO
FROM ubuntu:20.04 as stablehlo-builder
COPY --from=llvm-builder /stablehlo /stablehlo
RUN apt-get update && \
    apt-get install -y wget ninja-build lld clang ccache
RUN wget https://github.com/Kitware/CMake/releases/download/v3.31.5/cmake-3.31.5-linux-x86_64.sh && \
    bash cmake-3.31.5-linux-x86_64.sh --skip-license --prefix=/opt && \
    ln -s /opt/bin/cmake /usr/local/bin/cmake 
WORKDIR /stablehlo
RUN export CC=clang && \
    export CXX=clang++ && \
    mkdir -p build && cd build && \
    cmake .. -GNinja \
	  -DLLVM_ENABLE_LLD="$LLVM_ENABLE_LLD" \
	  -DCMAKE_BUILD_TYPE=Release \
	  -DLLVM_ENABLE_ASSERTIONS=ON \
	  -DSTABLEHLO_ENABLE_BINDINGS_PYTHON=OFF \
	  -DMLIR_DIR=${PWD}/../llvm-build/lib/cmake/mlir && \
    cmake --build .
WORKDIR /stablehlo/build
RUN ninja check-stablehlo-tests

# STAGE 4: Packing
FROM ubuntu:20.04

# StableHLO
COPY --from=stablehlo-builder /stablehlo/build/bin /opt/stablehlo/bin
COPY --from=stablehlo-builder /stablehlo/build/lib /opt/stablehlo/lib

# Environment
RUN echo "export PATH=/opt/stablehlo/bin:$PATH" >> ~/.bashrc
WORKDIR /working_dir
