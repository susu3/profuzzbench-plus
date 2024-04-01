FROM ubuntu:20.04

# Install common dependencies
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -y update && \
    apt-get -y install sudo \ 
    apt-utils \
    build-essential \
    openssl \
    clang \
    graphviz-dev \
    git \
    automake \
    autoconf \
    libtool \
    libgnutls28-dev \
    libssl-dev \
    llvm \
    python3-pip \
    nano \
    net-tools \
    vim \
    gdb \
    netcat \
    strace \
    wget

# Add a new user ubuntu, pass: ubuntu
RUN groupadd ubuntu && \
    useradd -rm -d /home/ubuntu -s /bin/bash -g ubuntu -G sudo -u 1000 ubuntu -p "$(openssl passwd -1 ubuntu)"

RUN chmod 777 /tmp

RUN pip3 install gcovr==4.2

# Use ubuntu as default username
USER ubuntu
WORKDIR /home/ubuntu

# Import environment variable to pass as parameter to make (e.g., to make parallel builds with -j)
ARG MAKE_OPT

# Set up fuzzers
RUN git clone https://github.com/susu3/aflnet.git && \
    cd aflnet && \
    make clean all $MAKE_OPT && \
    cd llvm_mode && make $MAKE_OPT

RUN git clone https://github.com/susu3/aflnwe.git && \
    cd aflnwe && \
    make clean all $MAKE_OPT && \
    cd llvm_mode && make $MAKE_OPT

# Set up environment variables for AFLNet
ENV WORKDIR="/home/ubuntu/experiments"
ENV AFLNET="/home/ubuntu/aflnet"
ENV PATH="${PATH}:${AFLNET}:/home/ubuntu/.local/bin:${WORKDIR}"
ENV AFL_PATH="${AFLNET}"
ENV AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES=1 \
    AFL_SKIP_CPUFREQ=1 \
    AFL_NO_AFFINITY=1


RUN mkdir $WORKDIR

COPY --chown=ubuntu:ubuntu fuzzing.patch ${WORKDIR}/fuzzing.patch
COPY --chown=ubuntu:ubuntu gcov.patch ${WORKDIR}/gcov.patch

# Set up environment variables for ASAN
env ASAN_OPTIONS='abort_on_error=1:symbolize=0:detect_leaks=0:detect_stack_use_after_return=1:detect_container_overflow=0:poison_array_cookie=0:malloc_fill_byte=0:max_malloc_fill_size=16777216'

# Download and compile libmodbus for fuzzing
RUN cd $WORKDIR && \
    git clone https://github.com/stephane/libmodbus.git && \
    cd libmodbus && \
    git checkout 5c14f13 && \
    patch -p1 < ${WORKDIR}/fuzzing.patch && \
    ./autogen.sh && \
    ./configure && \
    AFL_USE_ASAN=1 CC=afl-clang-fast make clean all $MAKE_OPT

# Set up libmodbus for fuzzing
# ...libmodbus need no config file???
RUN cd ${WORKDIR}/libmodbus/tests 

# Download and compile libmodbus for coverage analysis
RUN cd $WORKDIR && \
    git clone https://github.com/stephane/libmodbus.git libmodbus-cov && \
    cd libmodbus-cov && \
    git checkout 5c14f13 && \
    patch -p1 < ${WORKDIR}/gcov.patch && \
    ./autogen.sh && \
    ./configure && \
    make CFLAGS="-fprofile-arcs -ftest-coverage" CPPFLAGS="-fprofile-arcs -ftest-coverage" CXXFLAGS="-fprofile-arcs -ftest-coverage" LDFLAGS="-fprofile-arcs -ftest-coverage" clean all $MAKE_OPT

# Set up libmodbus for coverage analysis
RUN cd ${WORKDIR}/libmodbus-cov/tests


COPY --chown=ubuntu:ubuntu in-modbus ${WORKDIR}/in-modbus
COPY --chown=ubuntu:ubuntu modbus.dict ${WORKDIR}/modbus.dict
COPY --chown=ubuntu:ubuntu cov_script.sh ${WORKDIR}/cov_script
COPY --chown=ubuntu:ubuntu run.sh ${WORKDIR}/run

