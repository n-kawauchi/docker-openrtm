FROM ubuntu:20.04

LABEL maintainer="Noriaki Ando <n-ando@aist.go.jp>"

ENV DEBIAN_FRONTEND=noninteractive

# initial apt update and installing wget, omniidl, python3 
RUN apt-get update \
    && apt-get install -y wget python3-pip \
    && mkdir -p /etc/apt/keyrings \
    && wget --secure-protocol=TLSv1_2 --no-check-certificate https://openrtm.org/pub/openrtm.key -O /etc/apt/keyrings/openrtm.key \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/openrtm.key] http://openrtm.org/pub/Linux/ubuntu/ $(cat /etc/os-release | sed -n 's/VERSION_CODENAME=//p') main" > /etc/apt/sources.list.d/openrtm.list \
    && apt-get update \
    && apt-get install -y omniidl-python3 openrtm2-python3 \
    && python3 -m pip install rtshell-aist \
    && apt-get purge -y --auto-remove wget \
    && rm -rf /var/lib/apt/lists/*

CMD ["/bin/bash"]
