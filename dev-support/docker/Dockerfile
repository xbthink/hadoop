# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Dockerfile for installing the necessary dependencies for building Hadoop.
# See BUILDING.txt.

FROM ubuntu:focal@sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322

WORKDIR /root

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

#####
# Disable suggests/recommends
#####
RUN echo APT::Install-Recommends "0"\; > /etc/apt/apt.conf.d/10disableextras
RUN echo APT::Install-Suggests "0"\; >>  /etc/apt/apt.conf.d/10disableextras

ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_TERSE true

######
# Platform package dependency resolver
######
COPY pkg-resolver/*.py pkg-resolver/
COPY pkg-resolver/*.json pkg-resolver/
RUN chmod a+x pkg-resolver/*.py \
    && chmod a+r pkg-resolver/*.json

######
# Install packages from apt
######
# hadolint ignore=DL3008,SC2046
RUN apt-get -q update \
    && apt-get -q install -y --no-install-recommends python3 \
    && apt-get -q install -y --no-install-recommends $(pkg-resolver/resolve.py ubuntu:focal) \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.UTF-8
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8'
ENV PYTHONIOENCODING=utf-8

######
# Set env vars required to build Hadoop
######
ENV MAVEN_HOME /usr
# JAVA_HOME must be set in Maven >= 3.5.0 (MNG-6003)
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

#######
# Set env vars for SpotBugs 4.2.2
#######
ENV SPOTBUGS_HOME /opt/spotbugs

#######
# Set env vars for Google Protobuf 3.7.1
#######
ENV PROTOBUF_HOME /opt/protobuf
ENV PATH "${PATH}:/opt/protobuf/bin"

###
# Avoid out of memory errors in builds
###
ENV MAVEN_OPTS -Xms256m -Xmx3072m

# Skip gpg verification when downloading Yetus via yetus-wrapper
ENV HADOOP_SKIP_YETUS_VERIFICATION true

####
# Install packages
####
COPY pkg-resolver/install-common-pkgs.sh pkg-resolver/
RUN bash pkg-resolver/install-common-pkgs.sh
COPY pkg-resolver/install-spotbugs.sh pkg-resolver/
RUN bash pkg-resolver/install-spotbugs.sh ubuntu:focal
COPY pkg-resolver/install-boost.sh pkg-resolver/


COPY boost_1_72_0.tar.bz2 boost_1_72_0.tar.bz2
RUN bash pkg-resolver/install-boost.sh ubuntu:focal
COPY pkg-resolver/install-protobuf.sh pkg-resolver/
RUN bash pkg-resolver/install-protobuf.sh ubuntu:focal
COPY pkg-resolver/install-hadolint.sh pkg-resolver/
RUN bash pkg-resolver/install-hadolint.sh ubuntu:focal
COPY pkg-resolver/install-intel-isa-l.sh pkg-resolver/
RUN bash pkg-resolver/install-intel-isa-l.sh ubuntu:focal

###
# Everything past this point is either not needed for testing or breaks Yetus.
# So tell Yetus not to read the rest of the file:
# YETUS CUT HERE
###

# Add a welcome message and environment checks.
COPY hadoop_env_checks.sh /root/hadoop_env_checks.sh
RUN chmod 755 /root/hadoop_env_checks.sh
# hadolint ignore=SC2016
RUN echo '${HOME}/hadoop_env_checks.sh' >> /root/.bashrc
