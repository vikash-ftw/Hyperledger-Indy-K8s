# ==========================================
# STAGE 1: The Downloader Sandbox
# ==========================================
FROM ubuntu:20.04 AS downloader

ENV DEBIAN_FRONTEND=noninteractive
# Install heavy utilities we DO NOT want in our production image
RUN apt-get update && apt-get install -y wget unzip tar

WORKDIR /downloads

# 1. Fetch System Debs
RUN wget -q http://security.ubuntu.com/ubuntu/pool/main/o/openssl1.0/libssl1.0.0_1.0.2n-1ubuntu5.13_amd64.deb && \
    wget -q https://hyperledger.jfrog.io/artifactory/indy/pool/bionic/master/u/ursa/ursa_0.3.2-1_amd64.deb

# 2. Fetch & Extract Plenum
RUN wget -q https://github.com/hyperledger/indy-plenum/releases/download/v1.13.1/third-party-dependencies.zip && \
    unzip -q third-party-dependencies.zip -d plenum-deps && \
    wget -q https://github.com/hyperledger/indy-plenum/releases/download/v1.13.1/plenum-deb.zip && \
    unzip -q plenum-deb.zip -d plenum

# 3. Fetch & Extract Node
RUN wget -q https://github.com/hyperledger/indy-node/releases/download/v1.13.2/third-party-dependencies.zip -O node-deps.zip && \
    unzip -q node-deps.zip -d node-deps && \
    wget -q https://github.com/hyperledger/indy-node/releases/download/v1.13.2/indy_node-deb.zip && \
    unzip -q indy_node-deb.zip -d indy_node

# 4. Fetch & Extract Admin CLI
RUN wget -q https://github.com/hyperledger/indy-cli-rs/releases/download/v0.1.0/indy-cli-rs-0.1.0-linux-x86_64.tar.gz && \
    tar -xf indy-cli-rs-0.1.0-linux-x86_64.tar.gz

# ==========================================
# STAGE 2: The Lean Production Image
# ==========================================
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

# 1. Install Base Runtime & Repositories
RUN apt-get update && apt-get install -y software-properties-common && \
    add-apt-repository universe && \
    apt-get update && apt-get install -y \
    gnupg apt-transport-https lsb-release \
    python3 python3-pip python3-setuptools \
    net-tools iproute2 nano dnsutils jq \
    python3-zipp python3-pytest python3-attr python3-more-itertools \
    python3-pluggy python3-py python3-atomicwrites python3-pyparsing python3-wcwidth \
    python3-dev libgflags-dev libsnappy-dev libbz2-dev liblz4-dev zlib1g-dev libc6-dev && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 9692C00E657DDE61 && \
    echo "deb https://hyperledger.jfrog.io/artifactory/indy focal stable" | tee /etc/apt/sources.list.d/indy.list && \
    apt-get update

# 2. Copy ONLY the extracted payloads from Stage 1
COPY --from=downloader /downloads/libssl1.0.0_1.0.2n-1ubuntu5.13_amd64.deb /tmp/
COPY --from=downloader /downloads/ursa_0.3.2-1_amd64.deb /tmp/
COPY --from=downloader /downloads/plenum-deps/artifacts/third-party-dependencies/*.deb /tmp/plenum-deps/
COPY --from=downloader /downloads/plenum/artifacts/plenum-deb/*.deb /tmp/plenum/
COPY --from=downloader /downloads/node-deps/artifacts/third-party-dependencies/*.deb /tmp/node-deps/
COPY --from=downloader /downloads/indy_node/artifacts/indy_node-deb/*.deb /tmp/indy_node/
COPY --from=downloader /downloads/indy-cli-rs /usr/local/bin/indy-cli

# 3. The "Mega-Layer" (Patched for Docker systemd limitations)
# Using 'apt-get install' on local files prevents silent uninstallations.
RUN ln -s /bin/true /usr/local/bin/systemctl && \
    apt-get install -y /tmp/libssl1.0.0_1.0.2n-1ubuntu5.13_amd64.deb /tmp/ursa_0.3.2-1_amd64.deb && \
    mv /usr/lib/ursa/* /usr/lib || true && \
    apt-get install -y /tmp/plenum-deps/*.deb && \
    apt-get install -y /tmp/plenum/*.deb && \
    apt-get install -y /tmp/node-deps/*.deb && \
    apt-get install -y /tmp/indy_node/*.deb && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/*

# 4. The Temporary Safety Net
RUN mkdir -p /etc/indy && \
    echo "NETWORK_NAME = 'build_test'" > /etc/indy/indy_config.py && \
    generate_indy_pool_transactions --help && \
    rm /etc/indy/indy_config.py  # remove the config to pass config using configmap

# 5. Setup Indy User
RUN id indy &>/dev/null || useradd -ms /bin/bash indy

EXPOSE 9701 9702