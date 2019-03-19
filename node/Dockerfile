# Use an official Python runtime as a parent image
FROM ubuntu:16.04
LABEL maintainer "Julian Janda <julian.janda@aau.at>"
ARG VERSION_CXX=ndn-cxx-0.6.0 
ARG VERSION_NFD=NFD-0.6.0
ARG VERSION_NDNCPP=v0.15
ARG BRANCH_NDNRTC=arc-dev_2.1
ARG USER=nfd
ARG PASS=nfd

# Copy ssh stuff for gitlab access
COPY config root/.ssh/config 
COPY id_rsa root/.ssh/id_rsa
COPY id_rsa.pub root/.ssh/id_rsa.pub

# Install any needed packages
RUN apt-get update \
    && apt-get -y install git sudo openssh-server build-essential libssl-dev

# Install sshd and add ssh fingerprint to known hosts
RUN mkdir /var/run/sshd \
    && chmod 0755 /var/run/sshd \
    && /usr/sbin/sshd \
    && useradd --create-home --shell /bin/bash --groups sudo $USER \
    && echo $USER':'$PASS | chpasswd \
    && ssh-keyscan -H gitlab.itec.aau.at >> ~/.ssh/known_hosts


# ========= NFD =========
# install tools
RUN  apt-get update \
     && apt-get install -y git build-essential

# install ndn-cxx and NFD dependencies
RUN apt-get install -y python libsqlite3-dev libboost-all-dev libssl-dev pkg-config libssl-dev libpcap-dev

# install ndn-cxx
RUN git clone https://github.com/named-data/ndn-cxx.git \
    && cd ndn-cxx \
    && git checkout $VERSION_CXX \
    && ./waf configure \
    && ./waf \
    && ./waf install \
    && cd .. \
    && rm -Rf ndn-cxx \
    && ldconfig

# install NFD
RUN git clone --recursive https://github.com/named-data/NFD \
    && cd NFD \
    && git checkout $VERSION_NFD \
    && ./waf configure \
    && ./waf \
    && ./waf install \
    && cd .. \
    && rm -Rf NFD

# install python3
RUN apt-get install -y software-properties-common

# # install ndn-tools
# RUN git clone --recursive https://github.com/named-data/ndn-tools.git \
#     && cd ndn-tools \
#     && ./waf configure \
#     && ./waf \
#     && ./waf install \
#     && cd .. \
#     && rm -Rf ndn-tools

# initial configuration
RUN cp /usr/local/etc/ndn/nfd.conf.sample /usr/local/etc/ndn/nfd.conf \
    && ndnsec-keygen /`whoami` | ndnsec-install-cert - \
    && mkdir -p /usr/local/etc/ndn/keys \
    && ndnsec-cert-dump -i /`whoami` > default.ndncert \
    && mv default.ndncert /usr/local/etc/ndn/keys/default.ndncert

RUN mkdir /share \
    && mkdir /logs

# cleanup
RUN apt autoremove \
    && apt-get remove -y git build-essential python pkg-config

# ========= NDN-RTC =========

# Protobuff bug workaround
RUN apt-get remove libprotobuf-dev

# RUN  apt-get update \
#      && apt-get install libboost-all-dev cmake wget autoconf automake libtool git protobuf-compiler

# Installing prerequisites
RUN  apt-get update \
     && apt-get install -y libssl-dev libboost-all-dev libprotobuf-dev libsqlite3-dev sudo \
     libconfig++9v5 libconfig++-dev wget autoconf automake libtool cmake git build-essential \
     lsb-release protobuf-compiler gawk


# Installing NDN-CPP
RUN cd / \
    && git clone https://github.com/named-data/ndn-cpp \
    && cd ndn-cpp \
    && git checkout $VERSION_NDNCPP \
    && ./configure --with-std-shared-ptr=no --with-std-function=no \
    && make && make install \
    && rm -Rf /ndn-cpp
    

# Installing OpenFEC
RUN cd / \
    && wget http://openfec.org/files/openfec_v1_4_2.tgz \
    && tar -xvf openfec_v1_4_2.tgz && rm openfec_v1_4_2.tgz \
    && mkdir -p openfec_v1.4.2/build && cd openfec_v1.4.2/ \
    && wget https://raw.githubusercontent.com/remap/ndnrtc/master/cpp/resources/ndnrtc-openfec.patch \
    && patch src/CMakeLists.txt ndnrtc-openfec.patch \
    && cd build/ \
    && cmake .. -DDEBUG:STRING=OFF \
    && make

# # Installing libconfig
# RUN cd / \
#     && git clone https://github.com/hyperrealm/libconfig.git \
#     && cd libconfig \
#     && mkdir -p build/share \
#     && autoreconf -i . \
#     && ./configure --prefix=$(pwd)/build \
#     && make && make install

# WebRTC & NDN-RTC (doing in one step to avoid large image size)
RUN git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git \
    && export PATH=$PATH:`pwd`/depot_tools \
    && mkdir webrtc-checkout && cd webrtc-checkout \
    && fetch --nohooks webrtc \
    && cd src && ./build/install-build-deps.sh --no-syms --no-prompt \
    && git checkout -b branch-heads-59 refs/remotes/branch-heads/59 \
    && gclient sync \
    && gn gen out/Default --args='is_debug=false' \
    && ninja -C out/Default \
    && cd / \
    && git clone --recursive https://github.com/jujanda/ndnrtc_arc.git \
    && cd ndnrtc_arc/cpp/ && mkdir -p build/share \
    && git checkout $BRANCH_NDNRTC \
    && echo 'CPPFLAGS="-DWEBRTC_POSIX" CXXFLAGS="-DWEBRTC_POSIX"' > build/share/config.site \
    && echo OPENFECDIR=/openfec_v1.4.2 >> build/share/config.site \
    && echo WEBRTCDIR=/webrtc-checkout/src >> build/share/config.site \
    && ./configure --prefix=$(pwd)/build \
    && make && make install && make ndnrtc-client \
    && rm -Rf /webrtc-checkout \
    && rm -Rf /depot_tools

# # Install prerequisites for headless client app 
# RUN apt-get update \
#     && apt-get install libxrender-dev \
#     && apt-get install libxcomposite-dev \
#     && make ndnrtc-client


# Install command scheduler
RUN git clone https://github.com/theuerse/emulation-cmdscheduler.git \
    && cd emulation-cmdscheduler/ \
    && cmake . \
    && make \
    && make install


# cleanup
RUN apt autoremove \
    && apt-get remove -y wget autoconf automake libtool cmake git build-essential \
                         lsb-release protobuf-compiler

# # Create identity and install certificate
# RUN ndnsec-keygen /ndndn/test | ndnsec-install-cert - \
#     && ndnsec-dump-certificate -i /ndndn/test > /ndnrtc/cpp/tests/policy_config/signing.cert

# Create identity and install certificate (move upwards)
RUN ndnsec-keygen /ndnrtc_arc | ndnsec-install-cert - \
    && ndnsec-dump-certificate -i /ndnrtc_arc > /ndnrtc_arc/cpp/tests/policy_config/signing.cert

# Install necessary tools fo emulation framework (move upwards)
RUN apt-get install -y nano iptables iperf
 

# (move upwards)
RUN mkdir /ndnrtc_arc/cpp/loopback

WORKDIR /ndnrtc_arc/cpp

# COPY producer.cfg /configs/
# COPY consumer.cfg /configs/
# COPY run.sh /ndnrtc/cpp
# RUN chmod +x /ndnrtc/cpp/run.sh

COPY in_256x144.raw /ndnrtc_arc/cpp/tests/
COPY sudoers /etc/sudoers

EXPOSE 6363/tcp
EXPOSE 6363/udp
EXPOSE 80

# Define environment variable
ENV NAME World
# ENV CONFIG=/usr/local/etc/ndn/nfd.conf
# ENV LOG_FILE=/logs/nfd.log

# CMD /usr/local/bin/nfd -c $CONFIG > $LOG_FILE 2>&1

# Run sshd when the container launches
CMD /usr/sbin/sshd -D && nfd-start -D