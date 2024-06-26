FROM debian:bullseye-slim

ENV VERSION 2021.9
ENV RELEASE_DATE 2021-09-05
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8
ENV DEBIAN_FRONTEND noninteractive

# Update base image and install dependencies.
RUN dpkg --add-architecture i386 \
    && apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        rsync \
        libc6:i386 \
        libstdc++6:i386\
    && rm -rf /var/lib/apt/lists/* \
    && groupadd -r steam && useradd -r -g steam -m -d /opt/steam steam \
    && mkdir /gamedir

USER steam
WORKDIR /opt/steam
COPY ./hldm.install /opt/steam

# Download SteamCMD and install HLDM.
RUN curl -sL media.steampowered.com/client/installer/steamcmd_linux.tar.gz | tar xzvf - \
    && ldd /opt/steam/linux32/steamcmd \
    && ./steamcmd.sh +runscript hldm.install \
    && rm -fr /opt/steam/hldm/cstrike \
    && rm -fr /opt/steam/hldm/siteserverui \
    && rm -fr /opt/steam/hldm/linux64

# Fix error that steamclient.so is missing.
RUN mkdir -p $HOME/.steam \
    && ln -s /opt/steam/linux32 $HOME/.steam/sdk32 \
    && echo 70 > /opt/steam/hldm/steam_appid.txt

WORKDIR /opt/steam/hldm

# Copy configs, Metamod, Stripper2 and AMX.
COPY --chown=steam:steam gamedir valve
COPY --chown=steam:steam ./entrypoint.sh ./entrypoint.sh

EXPOSE 27015
EXPOSE 27015/udp

# Start server.
ENTRYPOINT ["./entrypoint.sh", "-timeout 3"]

# Default start parameters.
CMD ["+maxplayers 12", "+map crossfire"]

# Labels
LABEL vendor=steamcalculator.com \
    hldm.docker.version="$VERSION" \
    hldm.docker.release-date="$RELEASE_DATE"
