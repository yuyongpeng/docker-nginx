FROM alpine:3.5

LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

ENV NGINX_VERSION 1.12.1
ENV NGINX_PATH /usr/local/nginx

RUN echo "https://mirrors.ustc.edu.cn/alpine/v3.6/main" > /etc/apk/repositories
RUN echo "https://mirrors.ustc.edu.cn/alpine/v3.6/community" >> /etc/apk/repositories
RUN apk update
RUN apk add --no-cache bash

RUN NGINX_PATH=/usr/local/nginx \
	&& mkdir -p $NGINX_PATH \
	&& GPG_KEYS=B0F4253373F8F6F510D42178520A9993A1C052F8 \
	&& CONFIG="\
		--prefix=$NGINX_PATH \
		--sbin-path=${NGINX_PATH} \
		--modules-path=${NGINX_PATH}/modules \
		--conf-path=${NGINX_PATH}/nginx.conf \
		--error-log-path=${NGINX_PATH}/error.log \
		--http-log-path=${NGINX_PATH}/access.log \
		--pid-path=${NGINX_PATH}/nginx.pid \
		--lock-path=${NGINX_PATH}/nginx.lock \
		--http-client-body-temp-path=${NGINX_PATH}/client_temp \
		--http-proxy-temp-path=${NGINX_PATH}/proxy_temp \
		--http-fastcgi-temp-path=${NGINX_PATH}/fastcgi_temp \
		--http-uwsgi-temp-path=${NGINX_PATH}/uwsgi_temp \
		--http-scgi-temp-path=${NGINX_PATH}/scgi_temp \
		--user=nginx \
		--group=nginx \
		--with-http_ssl_module \
		--with-http_realip_module \
		--with-http_addition_module \
		--with-http_sub_module \
		--with-http_dav_module \
		--with-http_flv_module \
		--with-http_mp4_module \
		--with-http_gunzip_module \
		--with-http_gzip_static_module \
		--with-http_random_index_module \
		--with-http_secure_link_module \
		--with-http_stub_status_module \
		--with-http_auth_request_module \
		--with-http_xslt_module=dynamic \
		--with-http_image_filter_module=dynamic \
		--with-http_geoip_module=dynamic \
		--with-threads \
		--with-stream \
		--with-stream_ssl_module \
		--with-stream_ssl_preread_module \
		--with-stream_realip_module \
		--with-stream_geoip_module=dynamic \
		--with-http_slice_module \
		--with-mail \
		--with-mail_ssl_module \
		--with-compat \
		--with-file-aio \
		--with-http_v2_module \
	" \
	&& addgroup -S nginx \
	&& adduser -D -S -h ${NGINX_PATH} -s /sbin/nologin -G nginx nginx \
	&& apk add --no-cache --virtual .build-deps \
		gcc \
		libc-dev \
		make \
		openssl-dev \
		pcre-dev \
		zlib-dev \
		linux-headers \
		curl \
		gnupg \
		libxslt-dev \
		gd-dev \
		geoip-dev \
	&& curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
	&& curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz.asc  -o nginx.tar.gz.asc \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& found=''; \
	for server in \
		ha.pool.sks-keyservers.net \
		hkp://keyserver.ubuntu.com:80 \
		hkp://p80.pool.sks-keyservers.net:80 \
		pgp.mit.edu \
	; do \
		echo "Fetching GPG key $GPG_KEYS from $server"; \
		gpg --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$GPG_KEYS" && found=yes && break; \
	done; \
	test -z "$found" && echo >&2 "error: failed to fetch GPG key $GPG_KEYS" && exit 1; \
	gpg --batch --verify nginx.tar.gz.asc nginx.tar.gz \
	&& rm -r "$GNUPGHOME" nginx.tar.gz.asc \
	&& mkdir -p /usr/src \
	&& tar -zxC /usr/src -f nginx.tar.gz \
	&& rm nginx.tar.gz \
	&& cd /usr/src/nginx-$NGINX_VERSION \
	&& ./configure $CONFIG --with-debug \
	# 获取CPU的核数
	# 允许一次运行的job数量
	&& make -j$(getconf _NPROCESSORS_ONLN) \
	&& mv objs/nginx objs/nginx-debug \
	&& mv objs/ngx_http_xslt_filter_module.so objs/ngx_http_xslt_filter_module-debug.so \
	&& mv objs/ngx_http_image_filter_module.so objs/ngx_http_image_filter_module-debug.so \
	&& mv objs/ngx_http_geoip_module.so objs/ngx_http_geoip_module-debug.so \
	&& mv objs/ngx_stream_geoip_module.so objs/ngx_stream_geoip_module-debug.so \
	&& ./configure $CONFIG \
	# 获取CPU的核数
	# 允许一次运行的job数量
	&& make -j$(getconf _NPROCESSORS_ONLN) \
	&& make install \
	&& rm -rf ${NGINX_VERSION}/html/ \
	&& mkdir -p $NGINX_PATH/conf.d/ \
	&& mkdir -p ${NGINX_VERSION}/html/ \
	# install 命令用于拷贝文件和设置变量
	&& install -m644 html/index.html ${NGINX_PATH}/html/ \
	&& install -m644 html/50x.html ${NGINX_PATH}/html/ \
	&& install -m755 objs/nginx-debug ${NGINX_PATH}/nginx-debug \
	&& install -m755 objs/ngx_http_xslt_filter_module-debug.so ${NGINX_PATH}/modules/ngx_http_xslt_filter_module-debug.so \
	&& install -m755 objs/ngx_http_image_filter_module-debug.so ${NGINX_PATH}/modules/ngx_http_image_filter_module-debug.so \
	&& install -m755 objs/ngx_http_geoip_module-debug.so ${NGINX_PATH}/modules/ngx_http_geoip_module-debug.so \
	&& install -m755 objs/ngx_stream_geoip_module-debug.so ${NGINX_PATH}/modules/ngx_stream_geoip_module-debug.so \
	# && ln -s ../../usr/lib/nginx/modules $NGINX_PATH/modules \
	&& strip $NGINX_PATH/sbin/nginx* \
	&& strip $NGINX_PATH/modules/*.so \
	&& rm -rf /usr/src/nginx-$NGINX_VERSION \
	\
	# Bring in gettext so we can get `envsubst`, then throw
	# the rest away. To do this, we need to install `gettext`
	# then move `envsubst` out of the way so `gettext` can
	# be deleted completely, then move `envsubst` back.
	&& apk add --no-cache --virtual .gettext gettext \
	&& mv /usr/bin/envsubst /tmp/ \
	\
	# 安装nginx的依赖包
	# scanelf 用户获得so文件所依赖的包，然后在解析字符串后，安装这些包
	&& runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' ${NGINX_PATH}/nginx ${NGINX_PATH}/modules/*.so /tmp/envsubst \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
	&& apk add --no-cache --virtual .nginx-rundeps $runDeps \
	&& apk del .build-deps \
	&& apk del .gettext \
	&& mv /tmp/envsubst /usr/local/bin/ \
	\
	# forward request and error logs to docker log collector
	&& ln -sf /dev/stdout $NGINX_PATH/access.log \
	&& ln -sf /dev/stderr $NGINX_PATH/error.log

COPY nginx.conf $NGINX_PATH/nginx.conf
COPY nginx.vh.default.conf $NGINX_PATH/conf.d/default.conf

EXPOSE 80

STOPSIGNAL SIGTERM

CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
