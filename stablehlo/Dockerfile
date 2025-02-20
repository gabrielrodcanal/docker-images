# STAGE 1: Build LLVM
FROM ubuntu:20.04 as llvm-builder

RUN apt-get update && \
    apt-get -y install wget git ninja-build lld clang && \
    apt-get clean -y && rm -rf /var/lib/apt/lists/*
RUN git clone --branch v1.9.2 --depth 1 https://github.com/openxla/stablehlo.git
RUN wget https://github.com/Kitware/CMake/releases/download/v3.31.5/cmake-3.31.5-linux-x86_64.sh && \
    bash cmake-3.31.5-linux-x86_64.sh --skip-license --prefix=/opt && \
    ln -s /opt/bin/cmake /usr/local/bin/cmake 

WORKDIR /stablehlo
RUN mkdir llvm-project && \
    cd llvm-project && \
    git init && \
    git remote add origin https://github.com/llvm/llvm-project.git && \
    git fetch --depth 1 origin $(cat ../build_tools/llvm_version.txt) && \
    git checkout FETCH_HEAD && \
    git submodule init && \
    git submodule update --depth=1

WORKDIR /stablehlo
RUN CC=clang CXX=clang++ MLIR_ENABLE_BINDINGS_PYTHON=OFF build_tools/build_mlir.sh $(pwd)/llvm-project/ $(pwd)/llvm-build

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
RUN ninja install

FROM ubuntu:20.04
COPY --from=llvm-builder /stablehlo/build/bin /usr/bin
COPY --from=llvm-builder /usr/local/lib /usr/local/lib
WORKDIR /workdir