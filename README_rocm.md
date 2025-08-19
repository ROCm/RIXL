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
## nixlbench

Only the UCX backend is support with ROCm in nixlbench as of today. The equivalent of the NVSHMEM backend using rocSHMEM will be developed in the future. In addition, support for `HIP_MEM_HANDLE_TYPE_FABRIC` is also not available with the current commit.

### Build & install
```
$ cd benchmark/nixlbench
$ meson setup build -Dnixl_path=${_nixl_install_dir} -Dcudapath_inc=/opt/rocm/include -Dcudapath_lib=/opt/rocm/lib --prefix=${_nixlbench_install_dir}
$ cd build
$ ninja
$ ninja install
```

### Python Interface
NIXL provides Python bindings through pybind11. For detailed Python API documentation, see [docs/python_api.md](docs/python_api.md).
You can build it from source :

```bash
# From the root nixl directory
pip install . --config-settings=setup-args="-Dcudapath_inc=/opt/rocm/include" --config-settings=setup-args="-Dcudapath_lib=/opt/rocm/lib" --config-settings=setup-args="-Ducx_path="${_ucx_install_dir}" --config-settings=setup-args="-Ddisable_gds_backend=true"
```

### Run a testcase
Running a nixlbench testcase in two separate windows.

Window 1:
```
$ export LD_LIBRARY_PATH=${_nixl_install_dir}/lib/x86_64-linux-gnu/:${_ucx_install_dir}/lib:/opt/rocm/lib
$ ./nixlbench --etcd-endpoints http://localhost:2379 --backend UCX --initiator_seg_type VRAM
```

Window 2:
```
$ export LD_LIBRARY_PATH=${_nixl_install_dir}/lib/x86_64-linux-gnu/:${_ucx_install_dir}/lib:/opt/rocm/lib
$ ./nixlbench --etcd-endpoints http://localhost:2379 --backend UCX --initiator_seg_type VRAM
```

The output in Window 1 should like approximatly as follows (please ignore the performance numbers):
```
WARNING: Adjusting num_iter to 1008 to allow equal distribution to 1 threads
WARNING: Adjusting warmup_iter to 112 to allow equal distribution to 1 threads
Connecting to ETCD at http://localhost:2379
ETCD Runtime: Registered as rank 0 item 1 of 2
Init nixl worker, dev all rank 0, type initiator, hostname hyd-7c-zt05-02
Waiting for all processes to start... (expecting 2 total: 1 initiators and 1 targets)
All processes are ready to proceed
**********************************************************************
NIXLBench Configuration
**********************************************************************
Runtime (--runtime_type=[etcd])                             : ETCD
ETCD Endpoint                                               : http://localhost:2379
Worker type (--worker_type=[nixl,nvshmem])                  : nixl
Backend (--backend=[UCX,UCX_MO,GDS,POSIX,OBJ])              : UCX
Enable pt (--enable_pt=[0,1])                               : 0
Device list (--device_list=dev1,dev2,...)                   : all
Enable VMM (--enable_vmm=[0,1])                             : 0
Initiator seg type (--initiator_seg_type=[DRAM,VRAM])       : VRAM
Target seg type (--target_seg_type=[DRAM,VRAM])             : DRAM
Scheme (--scheme=[pairwise,manytoone,onetomany,tp])         : pairwise
Mode (--mode=[SG,MG])                                       : SG
Op type (--op_type=[READ,WRITE])                            : WRITE
Check consistency (--check_consistency=[0,1])               : 0
Total buffer size (--total_buffer_size=N)                   : 8589934592
Num initiator dev (--num_initiator_dev=N)                   : 1
Num target dev (--num_target_dev=N)                         : 1
Start block size (--start_block_size=N)                     : 4096
Max block size (--max_block_size=N)                         : 67108864
Start batch size (--start_batch_size=N)                     : 1
Max batch size (--max_batch_size=N)                         : 1
Num iter (--num_iter=N)                                     : 1008
Warmup iter (--warmup_iter=N)                               : 112
Large block iter factor (--large_blk_iter_ftr=N)            : 16
Num threads (--num_threads=N)                               : 1
--------------------------------------------------------------------------------

Block Size (B)      Batch Size     Avg Lat. (us)  B/W (MiB/Sec)  B/W (GiB/Sec)  B/W (GB/Sec)
--------------------------------------------------------------------------------
4096                1              8.3631         467.082        0.456135       0.489771
8192                1              13.3036        587.248        0.573485       0.615774
16384               1              15.0347        1039.26        1.0149         1.08974
32768               1              16.0923        1941.93        1.89641        2.03626
65536               1              23.7222        2634.66        2.57291        2.76264
131072              1              39.2688        3183.18        3.10858        3.33781
262144              1              69.1081        3617.52        3.53273        3.79324
524288              1              1288.16        388.152        0.379054       0.407006
1048576             1              220.703        4530.97        4.42477        4.75106
2097152             1              453.714        4408.06        4.30475        4.62219
4194304             1              846.429        4725.74        4.61498        4.9553
8388608             1              1659.98        4819.32        4.70637        5.05343
16777216            1              3311.67        4831.4         4.71817        5.06609
33554432            1              6588.05        4857.28        4.74344        5.09323
67108864            1              13137.3        4871.61        4.75743        5.10826
```


