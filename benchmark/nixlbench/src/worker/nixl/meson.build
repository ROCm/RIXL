# SPDX-FileCopyrightText: Copyright (c) 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
#           Modifications Copyright (c) 2025 Advanced Micro Devices, Inc. All rights reserved.
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

hipify_cmd = find_program('hipify-perl')
hipify_src = generator(hipify_cmd,
                       output : ['@BASENAME@_hip.cpp'],
                       arguments : ['@INPUT@', '@EXTRA_ARGS@', '@OUTPUT@'])
nixl_worker_hip_src = hipify_src.process('nixl_worker.cpp', extra_args: '-o')

nixl_worker_sources = [
   nixl_worker_hip_src,
  'nixl_worker.h',
]

nixl_worker_deps = [openmp_dep, nixl_lib, nixl_build, nixl_serdes]
if cuda_available
  nixl_worker_deps += [cuda_dep]
endif

nixl_worker_lib = static_library('nixl_worker',
  nixl_worker_sources,
  include_directories: inc_dir,
  dependencies: nixl_worker_deps,
  install: true,
)