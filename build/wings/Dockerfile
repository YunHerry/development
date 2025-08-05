FROM debian:12-slim

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8
ENV PATH="/usr/local/go/bin:${PATH}"

USER root

# 修复1: 先安装 lsb-release（因为后面要用 lsb_release 命令）
RUN apt -y update \
    && apt -y --no-install-recommends install lsb-release ca-certificates

# 修复2: 修正所有版本号不匹配的问题
RUN apt -y update \
    && apt -y --no-install-recommends install software-properties-common gnupg2 curl sudo \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt -y update \
    && apt -y --no-install-recommends install \
        git \
        iputils-ping \
        tar \
        zip \
        nano \
        unzip \
        make \
        gcc \
        g++ \
        gdb \
        python3 \  # Debian 12 中 python 改为 python3
        docker-ce \
        docker-ce-cli \
        containerd.io \
        iproute2 \
        net-tools \
    && curl -OL "https://go.dev/dl/go1.21.8.linux-$(dpkg --print-architecture).tar.gz" \
    && tar xvf "go1.21.8.linux-$(dpkg --print-architecture).tar.gz" -C /usr/local \  # 修复版本号
    && chown -R root:root /usr/local/go \
    && rm -rf go*.gz \
    && /usr/local/go/bin/go install github.com/go-delve/delve/cmd/dlv@latest \
    && usermod -aG docker root

# 修复3: 正确设置环境变量（不要用 echo 追加到 bashrc）
ENV PATH="/usr/local/go/bin:$PATH"

EXPOSE 8080
EXPOSE 2022

WORKDIR /home/root/wings
