FROM alpine:3.5
MAINTAINER mitsugogo <mitsugogo@gmail.com>

RUN set -x \
	&& apk upgrade --update \
	&& apk add \
		bash \
		'nodejs>=6.5.0' \
		coreutils \
		libusb \
		pcsc-lite \
		pcsc-lite-libs \
		curl \
		ca-certificates \
	\
	&& apk add --virtual .build-deps \
		git \
		make \
		gcc \
		g++ \
		gzip \
		autoconf \
		automake \
		libc-dev \
		musl-dev \
		eudev-dev \
		libevent-dev \
		pcsc-lite-dev \
		libusb-dev \
	\
	&& npm install pm2 -g \
	&& npm install arib-b25-stream-test -g --unsafe \
	\
	&& touch /usr/local/bin/update-rc.d ; chmod a+x /usr/local/bin/update-rc.d \
	&& npm install mirakurun@latest -g --unsafe --production \
	\
	# recpt1
	&& git clone https://github.com/stz2012/recpt1 /tmp/recpt1 \
	&& cd /tmp/recpt1/recpt1 \
	&& ./autogen.sh \
	&& ./configure \
	&& sed -i '/#include <sys\/msg.h>/d' recpt1core.h \
	&& sed -i -E 's!(#include <sys/msg.h>)!#undef _GNU_SOURCE\n#undef _BSD_SOURCE\n\1!' recpt1.c \
	&& sed -i -E 's!(#include <sys/msg.h>)!#undef _GNU_SOURCE\n#undef _BSD_SOURCE\n\1!' recpt1ctl.c \
	&& sed -i -E 's!(#include <sys/msg.h>)!#undef _GNU_SOURCE\n#undef _BSD_SOURCE\n\1!' checksignal.c \
	&& sed -i -E 's!(#include <ctype.h>)!\1\n#include <event.h>!' tssplitter_lite.c \
	&& sed -i 's#-I../driver#-I../driver -I/usr/local/include#' Makefile \
	&& make \
	&& make install \
	\
	# ccid
	&& cd /tmp \
	&& curl -s https://alioth.debian.org/frs/download.php/file/4205/ccid-1.4.26.tar.bz2 -o ccid-latest.tar.bz2 \
	&& tar xvf ccid-*.tar.bz2 \
	&& cd ccid-* \
	&& ./configure \
	&& make \
	&& make install

# proxy
RUN mkdir /proxy \
        && cd /proxy \
        && git clone https://github.com/mitsugogo/BonDriverProxy_Linux ./BonDriverProxy_Linux \
        && cd BonDriverProxy_Linux \
        && make client \
        && cp BonDriver_Proxy.so BonDriver_Proxy-T0.so \
        && cp BonDriver_Proxy.so BonDriver_Proxy-T1.so \
        && cp BonDriver_Proxy.so BonDriver_Proxy-T2.so \
        && cp BonDriver_Proxy.so BonDriver_Proxy-T3.so \
        && cp BonDriver_Proxy.so BonDriver_Proxy-S0.so \
        && cp BonDriver_Proxy.so BonDriver_Proxy-S1.so \
        && cp BonDriver_Proxy.so BonDriver_Proxy-S2.so \
        && cp BonDriver_Proxy.so BonDriver_Proxy-S3.so

# COPY BonDriverProxy_conf
WORKDIR /proxy/BonDriverProxy_Linux/
COPY ./proxy_conf/BonDriver_Proxy-S.so.conf /proxy/BonDriverProxy_Linux/
COPY ./proxy_conf/BonDriver_Proxy-T.so.conf /proxy/BonDriverProxy_Linux/
RUN mv BonDriver_Proxy-S.so.conf BonDriver_Proxy-S0.so.conf \
        && cp BonDriver_Proxy-S0.so.conf BonDriver_Proxy-S1.so.conf \
        && cp BonDriver_Proxy-S0.so.conf BonDriver_Proxy-S2.so.conf \
        && cp BonDriver_Proxy-S0.so.conf BonDriver_Proxy-S3.so.conf 
RUN mv BonDriver_Proxy-T.so.conf BonDriver_Proxy-T0.so.conf \
        && cp BonDriver_Proxy-T0.so.conf BonDriver_Proxy-T1.so.conf \
        && cp BonDriver_Proxy-T0.so.conf BonDriver_Proxy-T2.so.conf \
        && cp BonDriver_Proxy-T0.so.conf BonDriver_Proxy-T3.so.conf

# recbond
RUN git clone https://github.com/mitsugogo/recbond.git /proxy/BonDriverProxy_Linux/recbond \
        && cd /proxy/BonDriverProxy_Linux/recbond \
        && ./autogen.sh \
        && ./configure \
        && make \
        && make install

# cleaning
# RUN cd / \
#	&& apk del --purge .build-deps \
#	&& rm -rf /tmp/* \
#	&& rm -rf /var/cache/apk/*

	# forward request and error logs to docker log collector
	#&& ln -sf /dev/stdout /usr/local/var/log/mirakurun.stdout-0.log \
	#&& ln -sf /dev/stderr /usr/local/var/log/mirakurun.stderr-0.log

WORKDIR /usr/local
COPY services.sh /usr/local/bin

CMD ["/usr/local/bin/services.sh"]
EXPOSE 40772
