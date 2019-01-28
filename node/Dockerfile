FROM ubuntu:16.04
LABEL maintainer "Julian Janda <julian.janda@aau.at>"
ARG VERSION_CXX=ndn-cxx-0.6.0 
ARG VERSION_NFD=NFD-0.6.0

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
# RUN  apt-get update \
#      && apt-get install libboost-all-dev cmake wget autoconf automake libtool git protobuf-compiler

# RUN  apt-get update \
#      && apt-get install -y libssl-dev libboost-all-dev libprotobuf-dev libsqlite3-dev sudo \
#      libconfig++9v5 libconfig++-dev wget autoconf automake libtool cmake git build-essential \
#      lsb-release protobuf-compiler gawk


# Installing NDN-CPP
# RUN cd / \
#     && git clone https://github.com/named-data/ndn-cpp \
#     && cd ndn-cpp \
#     && git checkout v0.13 \
#     && mkdir -p build/share \
#     && ./configure --with-std-shared-ptr=no --with-std-function=no --prefix=$(pwd)/build \
#     && make && make install \
#     && rm -Rf /ndn-cpp

# # Installing OpenFEC
# RUN cd / \
#     && wget http://openfec.org/files/openfec_v1_4_2.tgz \
#     && tar -xvf openfec_v1_4_2.tgz && rm openfec_v1_4_2.tgz \
#     && mkdir -p openfec_v1.4.2/build && cd openfec_v1.4.2/ \
#     && wget https://raw.githubusercontent.com/remap/ndnrtc/master/cpp/resources/ndnrtc-openfec.patch \
#     && patch src/CMakeLists.txt ndnrtc-openfec.patch \
#     && cd build/ \
#     && cmake .. -DDEBUG:STRING=OFF \
#     && make

# # Installing libconfig
# RUN cd / \
#     && git clone https://github.com/hyperrealm/libconfig.git \
#     && cd libconfig \
#     && mkdir -p build/share \
#     && autoreconf -i . \
#     && ./configure --prefix=$(pwd)/build \
#     && make && make install


# # Installing depot tools
# RUN cd / \
#     && git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git \
#     && export PATH=$PATH:`pwd`/depot_tools

# # Installing WebRTC (branch-heads/59)
# RUN mkdir webrtc-checkout \
#     && cd webrtc-checkout \
#     && fetch --nohooks webrtc \
#     && gclient sync \
#     && cd src \
#     && git checkout branch-heads/59 \
#     && gclient sync \
#     && ./build/install-build-deps.sh

#     # Installing NDN-RTC
# RUN && cd / \
#     && git clone --recursive https://github.com/remap/ndnrtc \
#     && cd ndnrtc \
#     && git checkout df267aa1c019a3a6644b7a78909345b639ceca8c \
#     && cd cpp && mkdir -p build/share \
#     && echo 'CPPFLAGS="-DWEBRTC_POSIX" CXXFLAGS="-DWEBRTC_POSIX"' >  build/share/config.site \
#     && echo NDNCPPDIR=`pwd`/../../ndn-cpp/build/include >> build/share/config.site \
#     && echo NDNCPPLIB=`pwd`/../../ndn-cpp/build/lib >> build/share/config.site \
#     && echo OPENFECDIR=`pwd`/../../openfec_v1.4.2 >> build/share/config.site \
#     && echo WEBRTCDIR=`pwd`/../../webrtc-checkout/src >> build/share/config.site \
#     && echo LCONFIGDIR=`pwd`/../../libconfig/build/include >> build/share/config.site \
#     && echo LCONFIGLIB=`pwd`/../../libconfig/build/lib >> build/share/config.site \
#     && ./configure --prefix=$(pwd)/build \
#     && make && make install

# # Install prerequisites for headless client app 
# RUN apt-get update \
#     && apt-get install libxrender-dev \
#     && apt-get install libxcomposite-dev \
#     && make ndnrtc-client

# # cleanup
# RUN apt autoremove \
#     && apt-get remove -y wget autoconf automake libtool cmake git build-essential \
#                          lsb-release protobuf-compiler

# # Create identity and install certificate
# RUN ndnsec-keygen /ndndn/test | ndnsec-install-cert - \
#     && ndnsec-dump-certificate -i /ndndn/test > /ndnrtc/cpp/tests/policy_config/signing.cert

# WORKDIR /ndnrtc/cpp

# COPY producer.cfg /configs/
# COPY consumer.cfg /configs/
# COPY run.sh /ndnrtc/cpp
# RUN chmod +x /ndnrtc/cpp/run.sh


EXPOSE 6363/tcp
EXPOSE 6363/udp
EXPOSE 22/tcp

ENV CONFIG=/usr/local/etc/ndn/nfd.conf
ENV LOG_FILE=/logs/nfd.log

CMD /usr/local/bin/nfd -c $CONFIG > $LOG_FILE 2>&1