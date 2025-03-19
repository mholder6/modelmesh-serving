FROM registry.redhat.io/openshift4/ose-cli:v4.14.0-202502191335.p0.g44b3ac2.assembly.stream.el8

# List of chromedriver versions for download - https://chromedriver.chromium.org/downloads
ARG CHROMEDRIVER_VER=104.0.5112.79
ARG ORG=opendatahub-io
ARG BRANCH=main
ARG ODS_CI_REPO=https://github.com/red-hat-data-services/ods-ci
ARG ODS_CI_GITREF=releases/1.7.0-5
ARG OC_CLI_URL=https://mirror.openshift.com/pub/openshift-v4/amd64/clients/ocp/latest/openshift-client-linux.tar.gz

ENV HOME /root
WORKDIR /root

RUN dnf install -y bc git go-toolset python3-pip unzip && \
    git clone https://github.com/crobby/peak $HOME/peak && \    
    cd $HOME/peak && \
    git submodule update --init

# install jq to help with parsing json
RUN curl -o /usr/local/bin/jq http://stedolan.github.io/jq/download/linux64/jq && \
    chmod +x /usr/local/bin/jq

RUN mkdir -p $HOME/src && \
    cd $HOME/src && \
    git clone --depth=1 --branch main https://github.com/${ORG}/modelmesh-serving && \
    # Clone ods-ci repo at specified git ref for the JupyterHub webUI tests
    git clone --depth=1 ${ODS_CI_REPO} ods-ci && cd ods-ci && \
    git fetch origin ${ODS_CI_GITREF} && git checkout FETCH_HEAD && \
    chmod -R 777 $HOME/src

COPY setup/operatorsetup scripts/install.sh scripts/installandtest.sh $HOME/peak/
COPY resources $HOME/peak/operator-tests/modelmesh/resources
COPY util $HOME/peak/operator-tests/modelmesh
COPY setup/kfctl_openshift.yaml $HOME/kfdef/
COPY basictests $HOME/peak/operator-tests/modelmesh/basictests

RUN chmod -R 777 $HOME/kfdef && \
    mkdir -p $HOME/.kube && \
    chmod -R 777 $HOME/.kube && \
    chmod -R 777 $HOME/peak && \
    mkdir -p /peak && \
    chmod -R 777 $HOME && \
    ln -s $HOME/peak/installandtest.sh /peak/installandtest.sh

# For local testing, you can add your own kubeconfig to the image
# Note:  Do not push the image to a public repo with your kubeconfig
#ADD kubeconfig /root/.kube/config
CMD $HOME/peak/installandtest.sh
