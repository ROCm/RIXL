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
hipify_cpp = generator(hipify_cmd,
                       output : ['@BASENAME@_hip.cpp'],
                       arguments : ['@INPUT@', '@EXTRA_ARGS@', '@OUTPUT@'])
utils_hip_cpp = hipify_cpp.process('utils.cpp', extra_args: '-o')

# need to figure out how to include the generated header file
#hipify_h = generator(hipify_cmd,
#                     output : ['@BASENAME@_hip.h'],
#                     arguments : ['@INPUT@', '@EXTRA_ARGS@', '@OUTPUT@'])
#utils_hip_h = hipify_h.process('utils.h', extra_args: '-o')

utils_sources = [
  utils_hip_cpp,
  'utils.h',
  'scope_guard.h'
]

utils_deps = [
  cuda_dep,
  gflags_dep,
  openmp_dep
]

utils_lib = static_library('utils',
  utils_sources,
  dependencies: utils_deps,
  include_directories: inc_dir
)
utils_dep = declare_dependency(
  link_with: utils_lib,
  dependencies: utils_deps,
  include_directories: inc_dir
)

