# Use an official Python runtime as a parent image
FROM ubuntu:16.04
ARG VERSION_NDNCPP=v0.15
ARG USER=nfd
ARG PASS=nfd

# Copy ssh stuff for gitlab access
COPY config root/.ssh/config 
COPY id_rsa root/.ssh/id_rsa
COPY id_rsa.pub root/.ssh/id_rsa.pub

# Set the working directory to /ndn
# WORKDIR /ndn

# Install any needed packages
RUN apt-get update \
	&& apt-get -y install git sudo openssh-server build-essential libssl-dev

# Install sshd and add ssh fingerprint to known hosts
RUN mkdir /var/run/sshd \
	&& chmod 0755 /var/run/sshd \
	&& /usr/sbin/sshd \
	&& useradd --create-home --shell /bin/bash --groups sudo $USER \
	&& echo $USER':'$PASS | chpasswd \
	&& ssh-keyscan -H gitlab.itec.aau.at,143.205.176.126 >> ~/.ssh/known_hosts
	# gitlab.itec.aau.at,143.205.176.126

RUN git clone --recursive https://github.com/jujanda/ndnrtc_arc.git \
	&& cd ndnrtc_arc/cpp/ \
	&& git checkout arc-dev_2.1

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD /usr/sbin/sshd -D