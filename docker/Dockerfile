FROM debian:bookworm-slim AS hydraplay
ARG TARGETARCH

#======================================= MOPIDY INSTALLATION ===================================================#
RUN apt-get update \
 && apt-get -y upgrade \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y ssl-cert \
    wget \
    curl \
    gcc  \
    gnupg \
    git \
    unzip \
    jq \
    ca-certificates \
    python3 \
    python3-pip \
    python3-cryptography \
    python3-setuptools \
    python3-pykka  \
    python3-gst-1.0 \
    gstreamer1.0-libav \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-tools \
    libgirepository1.0-dev \
 && mkdir -p /etc/apt/keyrings \
 && wget -q -O /etc/apt/keyrings/mopidy-archive-keyring.gpg https://apt.mopidy.com/mopidy.gpg \
 && wget -q -O /etc/apt/sources.list.d/mopidy.list https://apt.mopidy.com/bookworm.list \
 && apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y \
        mopidy \
        mopidy-soundcloud \
 && python3 -m pip install setuptools --break-system-packages \
 && python3 -m pip install Mopidy-Local --break-system-packages \
 && python3 -m pip install jinja2 tornado --break-system-packages \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* ~/.cache

#======================================= HYDRAPLAY INSTALLATION ==================================================#

RUN export LATEST_RELEASE_VER=$(curl -sL https://api.github.com/repos/mariolukas/HydraPlay/releases/latest | jq -r ".tag_name") \
 && export LATEST_RELEASE_ZIP=$(curl -sL https://api.github.com/repos/mariolukas/HydraPlay/releases/latest | jq -r ".assets[].browser_download_url") \
 && wget ${LATEST_RELEASE_ZIP} \
 && mkdir -p /tmp/hydraplay \
 && mkdir -p /app \
 && unzip hydraplay-${LATEST_RELEASE_VER}.zip "src/*" -d /tmp/hydraplay/ \
 && cp -R /tmp/hydraplay/src/* /app/. \
 && rm hydraplay-${LATEST_RELEASE_VER}.zip \
 && rm -R /tmp/hydraplay

#======================================= SNAPCAST INSTALLATION ===================================================#

ARG SNAPCASTVERSION=0.29.0
ARG TARGETARCH

COPY docker/entrypoint.sh /entrypoint.sh
COPY src/hydraplay.sh /app/hydraplay.sh

RUN export SNAPCAST_ARCH=$TARGETARCH \
 && if [ "$TARGETARCH" = "arm" ]; then SNAPCAST_ARCH=${TARGETARCH}hf; fi \
 && export SNAPCAST_POSTFIX=${SNAPCAST_ARCH} \
 && if [ "$SNAPCASTVERSION" = "0.28.0" ]; then SNAPCAST_POSTFIX=${SNAPCAST_ARCH}-bookworm; fi \
 && if [ "$SNAPCASTVERSION" = "0.29.0" ]; then SNAPCAST_POSTFIX=${SNAPCAST_ARCH}_bookworm; fi \
 && apt-get update \
 && apt-get install -y libavahi-client3 libavahi-common3 libatomic1 \
 && wget https://github.com/badaix/snapcast/releases/download/v${SNAPCASTVERSION}/snapserver_${SNAPCASTVERSION}-1_${SNAPCAST_POSTFIX}.deb \
 && dpkg -i --force-all snapserver_${SNAPCASTVERSION}-1_${SNAPCAST_POSTFIX}.deb \
 && apt-get -f install -y\
#
#========================================= FINAL STEPS ===========================================================#
#
 && mkdir -p /media \
 && mkdir -p /root/.config/snapcast/ \
 &&	touch /tmp/hydra.config.json \
 &&	chmod 664 /tmp/hydra.config.json \
 && chmod a+x /app/hydraplay.sh \
 && chmod a+x /entrypoint.sh \
 && apt-get remove -y  \
    wget \
    git \
    curl \
    jq \
    unzip \
    gcc \
    python3-pip \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* ~/.cache

#======================================= General Docker configs ===================================================#

EXPOSE 1705
EXPOSE 1704
EXPOSE 6670-6699
EXPOSE 8080
EXPOSE 80
EXPOSE 443

ENTRYPOINT ["/entrypoint.sh"]
