# ROCm NIXL library

This is the ROCm port of the NIXL library.
The original code can be found at https://github.com/ai-dynamo/nixl.

## Prerequisites for source build
### Ubuntu:
```
$ sudo apt install build-essential cmake pkg-config
$ sudo apt install libaio-dev liburing-dev

$ pip3 install meson==0.64.0
$ pip3 install "pybind11[global]"
```
NIXL was tested with UCX version 1.18.x. For ROCm builds, it is recommended to use the UCX version from `https://github.com/ROCm/ucx`

```
$ git clone https://github.com/ROCm/ucx -b v1.18.x
$ cd ucx
$ ./autogen.sh
$ mkdir build
$ cd build
$ ./configure                          \
    --enable-shared                    \
    --disable-static                   \
    --disable-doxygen-doc              \
    --enable-optimizations             \
    --enable-devel-headers             \
    --with-rocm=/opt/rocm              \
    --with-verbs                       \
    --with-dm                          \
    --enable-mt
$ make -j
```

### ETCD (Optional, but recommended)
NIXL can use ETCD for metadata distribution and coordination between nodes in distributed environments. To use ETCD with NIXL:
#### ETCD Server and Client
```
$ sudo apt install etcd etcd-server etcd-client
```

#### ETCD CPP API
Installed from https://github.com/etcd-cpp-apiv3/etcd-cpp-apiv3

```
$ sudo apt install libgrpc-dev libgrpc++-dev libprotobuf-dev protobuf-compiler-grpc
$ sudo apt install libcpprest-dev
$ git clone https://github.com/etcd-cpp-apiv3/etcd-cpp-apiv3.git
$ cd etcd-cpp-apiv3
$ mkdir build && cd build
$ cmake ..
$ make -j$(nproc)
$ sudo make install // will install in /usr/local by default
```

## Build & install

```
$ meson setup build/ --prefix=${_nixl_install_dir}
                     -Ducx_path=${_ucx_install_dir}
                     -Ddisable_gds_backend=true
                     -Dcudapath_inc=/opt/rocm/include
                     -Dcudapath_lib=/opt/rocm/lib

$ cd build
$ ninja
$ ninja-install
```
