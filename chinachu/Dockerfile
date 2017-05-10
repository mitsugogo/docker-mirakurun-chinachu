FROM alpine:3.5
MAINTAINER mitsugogo <mitsugogo@gmail.com>

ARG REPOSITORY="git://github.com/Chinachu/Chinachu.git"
ARG BRANCH="master"

RUN set -x \
	&& apk upgrade --update \
	&& apk add \
		bash \
		'nodejs>=6.2.0' \
		coreutils \
		curl \
		procps \
		ca-certificates \
	\
	&& apk add --virtual .build-deps \
		git \
		make \
		gcc \
		g++ \
		autoconf \
		automake \
		wget \
		curl \
		sudo \
		tar \
		xz \
		libc-dev \
		musl-dev \
		eudev-dev \
		libevent-dev \
	\
	&& adduser -D -u 1000 -h /usr/local/chinachu chinachu \
	&& mkdir -p /usr/local/chinachu \
	&& git clone ${REPOSITORY} /usr/local/chinachu \
	&& chown -R chinachu:chinachu /usr/local/chinachu \
	&& cd /usr/local/chinachu \
	&& git checkout ${BRANCH} \
	&& echo 1 | sudo -u chinachu ./chinachu installer \
	&& ./chinachu service operator  initscript | sudo -u chinachu tee /tmp/chinachu-operator \
	&& ./chinachu service wui initscript | sudo -u chinachu tee /tmp/chinachu-wui \
	&& sudo -u chinachu mkdir log \
	\
	&& chown root. /tmp/chinachu-operator /tmp/chinachu-wui \
	&& chmod u+x /tmp/chinachu-operator /tmp/chinachu-wui \
	&& mv /tmp/chinachu-operator /etc/init.d/ \
	&& mv /tmp/chinachu-wui /etc/init.d/ 

# Handbrake
RUN echo http://dl-cdn.alpinelinux.org/alpine/edge/community >> /etc/apk/repositories \
        && echo http://dl-cdn.alpinelinux.org/alpine/edge/main >> /etc/apk/repositories \
        && echo http://dl-cdn.alpinelinux.org/alpine/edge/testing >> /etc/apk/repositories \
        && echo http://dl-4.alpinelinux.org/alpine/v3.5/main/ >> /etc/apk/repositories
RUN apk update && apk add libass ffmpeg-libs libbluray musl libdvdnav libdvdread jansson libsamplerate libtheora libvorbis x264-libs x265 libxml2
RUN apk del patch && apk add patch
RUN apk upgrade -q -U -a && apk add -q -U php7 php7-gd
RUN apk add cmake lame-dev opus libxml2-dev yasm libtool
RUN apk add opus-dev jansson-dev libtheora-dev x264-dev libsamplerate-dev fribidi-dev harfbuzz-dev lame-dev libogg-dev libvorbis-dev linux-headers libass-dev

WORKDIR /tmp
RUN git clone https://github.com/HandBrake/HandBrake.git

RUN git clone https://github.com/lu-zero/mfx_dispatch.git mfx_dispatch-9f4a84d7 \
   && cd mfx_dispatch-9f4a84d7 \
   && git checkout 9f4a84d7 \
   && cd .. \
   && tar czvf mfx_dispatch-9f4a84d7.tar.gz mfx_dispatch-9f4a84d7 \
   && mkdir -p /tmp/HandBrake/download/ \
   && cp mfx_dispatch-9f4a84d7.tar.gz /tmp/HandBrake/download/

RUN echo '140.113.17.209  ftp.tw.freebsd.org' >> /etc/hosts



WORKDIR /tmp/HandBrake
RUN find /tmp/HandBrake/contrib/*/module.defs | xargs sed -i "s/http:\/\/download\.handbrake\.fr\/handbrake\/contrib\//http:\/\/ftp\.tw\.freebsd\.org\/distfiles\/handbrake\//g"
RUN find /tmp/HandBrake/contrib/*/module.defs | xargs sed -i "s/https:\/\/download\.handbrake\.fr\/handbrake\/contrib\//http:\/\/ftp\.tw\.freebsd\.org\/distfiles\/handbrake\//g"
RUN wget https://distfiles.macports.org/HandBrake/libdvdread-5.0.0-6-gcb1ae87.tar.gz -P /tmp/HandBrake/download/
RUN cd ./ \
#        && ./configure --enable-fdk-aac --enable-qsv --enable-x265 --enable-libav-aac --disable-gtk --launch-jobs=0 --launch \
        && ./configure --enable-fdk-aac --enable-libav-aac --disable-gtk --launch-jobs=0 --launch \
        && cd build \
        && make \
        && make install 

# Task Spooler
# コマンドをキューに
WORKDIR /tmp
RUN wget http://vicerveza.homeunix.net/~viric/soft/ts/ts-1.0.tar.gz \
       && tar zxvf ts-1.0.tar.gz \
       && cd ts-1.0 \
       && make \
       && make install 

# ts-renamer(改造版)
WORKDIR /
RUN apk add nodejs nodejs-npm \
       && git clone https://github.com/mitsugogo/ts-renamer.git \
       && cd ts-renamer \
       && npm i \
       && cp ./settings/settings.sample.json ./settings/settings.json

# cleaning 
# RUN cd / \
#	&& apk del --purge .build-deps \
#	&& rm -rf /tmp/* \
#	&& rm -rf /var/cache/apk/*i


# forward request and error logs to docker log collector
# RUN ln -sf /dev/stdout /usr/local/chinachu/log/operator \
#	&& ln -sf /dev/stdout /usr/local/chinachu/log/scheduler \
#	&& ln -sf /dev/stdout /usr/local/chinachu/log/wui


WORKDIR /usr/local/chinachu
RUN chmod 777 -R /usr/local/chinachu/log
COPY services.sh /usr/local/bin
COPY config.sample.json /usr/local/chinachu/config.sample.json

CMD ["/usr/local/bin/services.sh"]
EXPOSE 10772 20772 5353
