FROM ubuntu:20.04
LABEL MAINTAINER="machuan <machuan0228@sina.com>"
ARG RELEASE_NAME=focal
ARG ROOT_PASSWD=123456
ENV SSH_PORT 22
ENV PATH /root/miniconda3/bin:$PATH
#
# UTF-8 encoding to support chs characters
#
RUN echo "export LANG=C.UTF-8" >>/etc/profile \
&& apt-get update \
&& apt-get install -y ca-certificates \
#
# apt sources backup
#
&& cp /etc/apt/sources.list /etc/apt/sources.list.bak \
#
# switch apt source to TUNA
#
&& echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ release-name main restricted universe multiverse\n\
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ release-name-updates main restricted universe multiverse\n\
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ release-name-backports main restricted universe multiverse\n\
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ release-name-security main restricted universe multiverse" >/etc/apt/sources.list \
&& sed -i 's/release-name/'$RELEASE_NAME'/g' /etc/apt/sources.list \
#
# remove nvidia sources
#
&& rm -f /etc/apt/sources.list.d/cuda.list /etc/apt/sources.list.d/nvidia-ml.list \
#
# install necessary packages
#
&& apt-get update \
&& DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  apt-utils \
  vim \
  openssh-server \
  net-tools \
  iputils-ping \
  wget \
  curl \
  git \
  iptables \
  bzip2 \
  command-not-found \
#
# install DTP compile environment
#
    bash \
    g++ \
    cmake \
    make \
    libev-dev \
    libunwind-dev \
    libasan4 net-tools iproute2 iperf lsof python3 python3-pip \
    golang \
#
# download & install miniconda
#
&& wget -q https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh \
&& bash Miniconda3-latest-Linux-x86_64.sh -b \
&& rm Miniconda3-latest-Linux-x86_64.sh \
&& ln -s /root/miniconda3/etc/profile.d/conda.sh /etc/profile.d/conda.sh \
#
# switch conda source to TUNA
#
&& echo "channels:\n\
  - defaults\n\
show_channel_urls: true\n\
default_channels:\n\
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main\n\
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r\n\
custom_channels:\n\
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud\n\
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud\n\
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud\n\
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud\n\
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud" >/root/.condarc \
#
# create conda py37 environment
#
&& /root/miniconda3/bin/conda create -npy37 python=3.7 -y \
#
# switch pip source to TUNA & update pip
#
&& /root/miniconda3/envs/py37/bin/pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple \
&& /root/miniconda3/envs/py37/bin/pip install pip -U \
#
# enable root access with ssh: replace line starts with '#PermitRootLogin' or 'PermitRootLogin'
#
&& sed -i '/^#*PermitRootLogin/cPermitRootLogin yes' /etc/ssh/sshd_config \
#
# set root password
#
&& echo "root:$ROOT_PASSWD" | chpasswd \
#
# change ssh port & start ssh service when docker container starts
#
&& echo "#!/bin/bash\n\
sed -i \"s/Port 22/Port \$SSH_PORT/g\" /etc/ssh/sshd_config\n\
service ssh start\n\
/bin/bash" >/home/startup.sh \
&& chmod 777 /home/startup.sh \
#
# add welcome tips
#
&& CONDA_VERSION=`/root/miniconda3/condabin/conda -V` \
&& CONDA_VERSION=${CONDA_VERSION#* } \
&& echo "\
printf '\\\n'\n\
printf '\\\033[0;34m'\n\
printf 'Welcome to GPU server\\\n'\n\
printf 'Please change the default root password in time\\\n'\n\
printf '\\\n'\n\
printf ' CUDA:\\\t\\\tv$CUDA_VERSION\\\n'\n\
printf ' CUDNN:\\\t\\\tv$CUDNN_VERSION\\\n'\n\
printf ' Conda:\\\t\\\tMiniconda v$CONDA_VERSION  use \`conda activate py37\` to activate conda env\\\n'\n\
printf ' SSH port:\\\t$SSH_PORT for this container\\\n'\n\
printf '\\\033[0m'\n\
printf '\\\n'\n\
" >>/etc/update-motd.d/10-help-text \
#
# cleanup
#
&& rm -rf /var/lib/apt/lists/* /tmp/*
#
# create Rust environment
#
ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH \
    RUST_VERSION=1.55.0

RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='3dc5ef50861ee18657f9db2eeb7392f9c2a6c95c90ab41e45ab4ca71476b4338' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='67777ac3bc17277102f2ed73fd5f14c51f4ca5963adadf7f174adf4ebc38747b' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='32a1532f7cef072a667bac53f1a5542c99666c4071af0c9549795bbdb2069ec1' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='e50d1deb99048bc5782a0200aa33e4eea70747d49dffdc9d06812fd22a372515' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.24.3/${rustArch}/rustup-init"; \
    wget "$url"; \
    echo "${rustupSha256} *rustup-init" | sha256sum -c -; \
    chmod +x rustup-init; \
    ./rustup-init -y --no-modify-path --profile minimal --default-toolchain $RUST_VERSION --default-host ${rustArch}; \
    rm rustup-init; \
    chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version; \
    cargo --version; \
    rustc --version; \
    echo "[source.crates-io]\n\
    replace-with = 'tuna'\n\n\
    source.tuna]\n\
    registry = \"https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git\"" > $CARGO_HOME/config;

#
# set startup script
#
ENTRYPOINT /home/startup.sh
