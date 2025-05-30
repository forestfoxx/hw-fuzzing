# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Docker image for circuit verification

FROM hw-fuzzing/base-verilator
MAINTAINER trippel@umich.edu

# Setup directory structure
ENV HW=$SRC/hw
RUN mkdir $HW

# Copy over hardware fuzzing build scripts and common test bench code
COPY common.mk exe.mk $HW/
COPY hwfutils $HW/hwfutils
COPY tb $HW/tb
COPY scripts/ $SCRIPTS/
COPY opentitan-requirements.txt sim-requirements.txt $SRC/

# Install python dependencies
RUN apt-get update && apt-get install -y wget git xxd python3.8-dev
RUN rm /usr/bin/python3
RUN ln -s $(which python3.8) /usr/bin/python3

# Install OpenTitan and Python Dependencies
RUN python3 -m pip install -r $SRC/opentitan-requirements.txt
RUN python3 -m pip install -r $SRC/sim-requirements.txt

# Install HW Fuzzing Utils Dependencies
RUN python3 -m pip install -r $HW/hwfutils/requirements.txt
ENV PYTHONPATH="$PYTHONPATH:$HW/hwfutils"
RUN wget https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-linux-x86_64.sh -O /tmp/cmake.sh \
    && chmod +x /tmp/cmake.sh \
    && /tmp/cmake.sh --skip-license --prefix=/usr/local \
    && rm /tmp/cmake.sh

# Install kcov for post-fuzzing coverage tracing
RUN apt-get update && apt-get install -y \
      binutils-dev \
      libcurl4-openssl-dev \
      zlib1g-dev \
      libdw-dev \
      libiberty-dev \
      libssl-dev
RUN cd $SRC && git clone --depth=1 https://github.com/SimonKagstrom/kcov
RUN cd $SRC/kcov && mkdir build && cd build && cmake .. && make

# Remove uneeded packages and requirements files
RUN apt-get remove --purge -y git && apt-get autoremove -y
RUN rm $SRC/opentitan-requirements.txt $SRC/sim-requirements.txt
