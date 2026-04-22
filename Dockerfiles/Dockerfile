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

# 1. Install Base Runtime AND Build Dependencies
RUN apt-get update && apt-get install -y \
    gnupg apt-transport-https lsb-release software-properties-common \
    python3 python3-pip python3-setuptools \
    net-tools iproute2 nano dnsutils jq \
    python3-zipp python3-pytest python3-attr python3-more-itertools \
    python3-pluggy python3-py python3-atomicwrites python3-pyparsing python3-wcwidth \
    # These '-dev' packages are massive. We only need them momentarily for dpkg wheel compilation.
    python3-dev libgflags-dev libsnappy-dev libbz2-dev liblz4-dev zlib1g-dev libc6-dev && \
    # Add Indy Repo
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 9692C00E657DDE61 && \
    echo "deb https://hyperledger.jfrog.io/artifactory/indy focal stable" | tee /etc/apt/sources.list.d/indy.list && \
    apt-get update

# 2. Copy ONLY the extracted payloads from Stage 1 (Leaving wget/unzip/tar behind)
COPY --from=downloader /downloads/libssl1.0.0_1.0.2n-1ubuntu5.13_amd64.deb /tmp/
COPY --from=downloader /downloads/ursa_0.3.2-1_amd64.deb /tmp/
COPY --from=downloader /downloads/plenum-deps/artifacts/third-party-dependencies/*.deb /tmp/plenum-deps/
COPY --from=downloader /downloads/plenum/artifacts/plenum-deb/*.deb /tmp/plenum/
COPY --from=downloader /downloads/node-deps/artifacts/third-party-dependencies/*.deb /tmp/node-deps/
COPY --from=downloader /downloads/indy_node/artifacts/indy_node-deb/*.deb /tmp/indy_node/
COPY --from=downloader /downloads/indy-cli-rs /usr/local/bin/indy-cli

# 3. The "Mega-Layer": Install, Compile, and immediately PURGE to prevent caching bloat
RUN dpkg -i /tmp/libssl1.0.0_1.0.2n-1ubuntu5.13_amd64.deb || apt-get -f install -y && \
    dpkg -i /tmp/ursa_0.3.2-1_amd64.deb || apt-get -f install -y && \
    dpkg -i /tmp/plenum-deps/*.deb || apt-get install -yf && \
    dpkg -i /tmp/plenum/*.deb || apt-get install -yf && \
    mv /usr/lib/ursa/* /usr/lib || true && \
    dpkg -i /tmp/node-deps/*.deb || apt-get install -yf && \
    dpkg -i /tmp/indy_node/*.deb || apt-get install -yf && \
    apt-get install -f -y && \
    dpkg --configure -a && \
    # ==========================================
    # 🧹 CRITICAL CLEANUP: Slash Image Size Here
    # ==========================================
    apt-get purge -y python3-dev libgflags-dev libsnappy-dev libbz2-dev liblz4-dev zlib1g-dev libc6-dev && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/*

# 4. Setup Indy User
RUN id indy &>/dev/null || useradd -ms /bin/bash indy

EXPOSE 9701 9702