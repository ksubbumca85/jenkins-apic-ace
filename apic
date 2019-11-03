FROM jenkins/jenkins:lts-alpine

USER root

# variables for openshift client download
ENV OC_VERSION=v3.11.0 \
    OC_TAG_SHA=0cbc58b \
    BUILD_DEPS='tar gzip' \
    RUN_DEPS='curl ca-certificates gettext'

RUN apk --no-cache --allow-untrusted -X https://apkproxy.herokuapp.com/sgerrand/alpine-pkg-glibc add glibc glibc-bin

# Download Openshift 3.11 client and install
RUN apk --no-cache add $BUILD_DEPS $RUN_DEPS && \
    curl -sLo /tmp/oc.tar.gz https://github.com/openshift/origin/releases/download/${OC_VERSION}/openshift-origin-client-tools-${OC_VERSION}-${OC_TAG_SHA}-linux-64bit.tar.gz && \
    tar xzvf /tmp/oc.tar.gz -C /tmp/ && \
    mv /tmp/openshift-origin-client-tools-${OC_VERSION}-${OC_TAG_SHA}-linux-64bit/oc /usr/local/bin/ && \
    rm -rf /tmp/oc.tar.gz /tmp/openshift-origin-client-tools-${OC_VERSION}-${OC_TAG_SHA}-linux-64bit && \
    apk del $BUILD_DEPS

# install Python3
RUN apk add --no-cache python3 python3-dev curl gcc g++ && \
    ln -s /usr/include/locale.h /usr/include/xlocale.h && \
    python3 -m ensurepip && \
    rm -r /usr/lib/python*/ensurepip && \
    pip3 install --upgrade pip setuptools && \
    if [ ! -e /usr/bin/pip ]; then ln -s pip3 /usr/bin/pip ; fi && \
    if [[ ! -e /usr/bin/python ]]; then ln -sf /usr/bin/python3 /usr/bin/python; fi && \
    rm -r /root/.cache

# copy Python3 custommodule dependencies used in APIC/ACE Devops script
COPY requirements.txt /server_requirements.txt
RUN pip install -U pipenv && \
    pip install --extra-index-url=https://pypi.python.org/simple/ --no-cache-dir -r /server_requirements.txt

# copy Python3 custommodule dependencies used in APIC/ACE Devops script
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt

# Copy APIC toolkit executable file
COPY apic /usr/bin/apic

#install Telnet - for telnet test from jenkins to remote dependent servers like APIC/Docker registry etc . especially required to connect to Fyre services
RUN apk update
RUN apk add busybox-extras

USER jenkins