FROM ubuntu:20.04 as opencv-build

LABEL maintainer="Noriaki Ando <n-ando@aist.go.jp>"

ENV DEBIAN_FRONTEND=noninteractive

# OpenCV Version
ARG OPENCV_VER=4.8.1

ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# initial build dependencies
RUN sed -i -e 's/# deb-src/deb-src/g' /etc/apt/sources.list
RUN apt-get update
RUN apt -y build-dep opencv
RUN apt-get install -y --no-install-recommends \
  g++ \
  make \
  cmake \
  ca-certificates \
  wget \
  tzdata \
  && rm -rf /var/lib/apt/lists/*

# downloading OpenCV
RUN mkdir /root/opencv-${OPENCV_VER} /root/opencv_contrib-${OPENCV_VER}
RUN wget -O - https://github.com/opencv/opencv/archive/refs/tags/${OPENCV_VER}.tar.gz\
       | tar xfz - -C /root/opencv-${OPENCV_VER} --strip-components 1 \
    && wget -O - https://github.com/opencv/opencv_contrib/archive/refs/tags/${OPENCV_VER}.tar.gz\
       | tar xfz - -C /root/opencv_contrib-${OPENCV_VER} --strip-components 1

# build OpenCV
RUN cmake -DOPENCV_EXTRA_MODULES_PATH=/root/opencv_contrib-${OPENCV_VER}/modules \
  -DBUILD_opencv_java=OFF \
  -DCMAKE_INSTALL_PREFIX=/tmp/opencv/install \
  -S /root/opencv-${OPENCV_VER} \
  -B /tmp/opencv/build
RUN cmake --build /tmp/opencv/build  --target install/strip -- -j$(nproc)  

##### build ImageProcessing

FROM ubuntu:20.04 as imageprocessing-build

ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt update\
  && apt install -y --no-install-recommends \
  g++ \
  make \
  cmake \
  git \
  ca-certificates \
  wget \
  tzdata \
  python3 \
  python3-all-dev \
  python3-tk \
  tix-dev \
  build-essential \
  debhelper

RUN mkdir -p /etc/apt/keyrings \
  && wget --secure-protocol=TLSv1_2 --no-check-certificate https://openrtm.org/pub/openrtm.key -O /etc/apt/keyrings/openrtm.key \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/openrtm.key] http://openrtm.org/pub/Linux/ubuntu/ $(cat /etc/os-release | sed -n 's/VERSION_CODENAME=//p') main" > /etc/apt/sources.list.d/openrtm.list \
  && apt-get update \
  && apt-get install -y libomniorb4-dev \ 
  omniidl \
  omniidl-python3 \
  openrtm2-dev \
  openrtm2-python3 \
  sudo \
  libgtk-3-dev \
  libjpeg-dev \
  libopenexr-dev \
  libopenjp2-7 \
  libdc1394-22 \
  libgstreamer1.0-dev \
  libgstreamer-plugins-base1.0-dev \
  libavcodec-extra58 \
  libavformat-dev \
  libswscale-dev

COPY --from=opencv-build /tmp/opencv/install /usr/local/

WORKDIR /root
RUN git clone https://github.com/OpenRTM/ImageProcessing \
  && cmake -S /root/ImageProcessing/opencv -B/tmp/ip/build \
  && cmake --build /tmp/ip/build

WORKDIR /tmp/ip/build
RUN ./build_linux_package.sh
RUN mkdir ../deb-pkg \
  && cp *.deb ../deb-pkg

##### install ImageProcessing

FROM ubuntu:20.04 as imageprocessing-install

COPY --from=opencv-build /tmp/opencv/install /usr/local/
COPY --from=imageprocessing-build /tmp/ip/deb-pkg/ /root

ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
  && apt-get install -y wget \
  && mkdir -p /etc/apt/keyrings \
  && wget --secure-protocol=TLSv1_2 --no-check-certificate https://openrtm.org/pub/openrtm.key -O /etc/apt/keyrings/openrtm.key \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/openrtm.key] http://openrtm.org/pub/Linux/ubuntu/ $(cat /etc/os-release | sed -n 's/VERSION_CODENAME=//p') main" > /etc/apt/sources.list.d/openrtm.list \
  && apt-get update \
  && apt-get install -y omniidl \
  omniidl-python3 \
  openrtm2-dev \  
  sudo \
  libopenexr-dev \
  libdc1394-22 \
  libgstreamer1.0-dev \
  libgstreamer-plugins-base1.0-dev \
  libavcodec-extra58 \
  libavformat-dev \
  libswscale-dev \
  v4l-utils \
  qv4l2 \
  tzdata

WORKDIR /root
RUN debname=$(ls) \
  && dpkg -i ${debname} \
  && apt clean \
  && rm -rf /var/lib/apt/lists/*

CMD ["/bin/bash"]
