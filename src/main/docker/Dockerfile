#
# Copyright 2017-2022 Micro Focus or one of its affiliates.
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
#

ARG DOCKER_HUB_PUBLIC=docker.io
FROM ${DOCKER_HUB_PUBLIC}/opensuse/leap:latest

# Set the locale manually to a set inherited from the base image (defaults to POSIX)
ENV LANG=en_US.utf8

# Copy custom crypto-policy file
COPY DISABLE-CBC.pmod /etc/crypto-policies/policies/modules

# Copy in the home:alvistack public key
COPY home_alvistack_repo_key.pub .

# Update the OS packages, install cURL, postgreSQL client and dejavu-fonts
# Install crypto-policies-scripts and disable weaker security algorithms
RUN zypper -n refresh && \
    zypper -n update && \
    zypper addrepo https://download.opensuse.org/repositories/home:alvistack/openSUSE_Leap_15.4/home:alvistack.repo && \
    rpmkeys --import home_alvistack_repo_key.pub && \
    rm home_alvistack_repo_key.pub && \
    zypper -n refresh && \
    zypper -n install curl postgresql dejavu-fonts crypto-policies-scripts gosu && \
    update-crypto-policies --set DEFAULT:DISABLE-CBC && \
    sed -rie '/^jdk.tls.disabledAlgorithms=/s/$/, TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256/' \
        /etc/crypto-policies/back-ends/java.config && \
    zypper -n remove -u crypto-policies-scripts && \
    zypper removerepo home_alvistack && \
    zypper -n clean --all

# Add scripts to be executed during startup
COPY startup /startup
ADD https://raw.githubusercontent.com/CAFapi/caf-common/v1.19.0/container-cert-script/install-ca-cert.sh \
    /startup/startup.d/install-ca-cert-base.sh
RUN chmod +x /startup/* /startup/startup.d/*

# Add Tini
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--", "/startup/startup.sh"]

# Add other useful scripts
COPY scripts /scripts
RUN chmod +x /scripts/*
