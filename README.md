dockerfile-snippets
===================
# Edit a config file inline w/ sed (https://registry.hub.docker.com/u/jprjr/php-fpm/dockerfile/)
RUN sed -i 's/;extension=gd.so/extension=gd.so/g' /etc/php/php.ini
RUN sed -i '/^listen/c \
listen = 0.0.0.0:9000' /etc/php/php-fpm.conf



# Edit an entire directory with find and exec sed (from official apache)
RUN find "$APACHE_CONFDIR" -type f -exec sed -ri ' \
	s!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g; \
	s!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g; \
' '{}' ';'



RUN touch /var/log/php-fpm.log

RUN chown -R http /var/log/php-fpm.log /run/php-fpm

USER http

From official postgres
# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r postgres && useradd -r -g postgres postgres

# make the "en_US.UTF-8" locale so postgres will be utf-8 enabled by default
RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

# Sed editing of a config file inline with package installs
RUN apt-get update \
	&& apt-get install -y postgresql-common \
	&& sed -ri 's/#(create_main_cluster) .*$/\1 = false/' /etc/postgresql-common/createcluster.conf \
	&& apt-get install -y \
		postgresql-$PG_MAJOR=$PG_VERSION \
		postgresql-contrib-$PG_MAJOR=$PG_VERSION \
	&& rm -rf /var/lib/apt/lists/*

# mkdir + chown in one step	
RUN mkdir -p /var/run/postgresql && chown -R postgres /var/run/postgresql

official golang
ENV PATH /usr/src/go/bin:$PATH

official python
# remove several traces of debian python
RUN apt-get purge -y python python-minimal python2.7-minimal

from official redis
# save buildDeps for late; set -x for debug mode
RUN buildDeps='gcc libc6-dev make'; \
	set -x; \
	apt-get update && apt-get install -y $buildDeps --no-install-recommends \
	&& apt-get purge -y $buildDeps \
	

See here for version pinning for cache-busting:
https://github.com/nginxinc/docker-nginx/commit/d5b14ea77879ca2b07293eacd16bbc20a0aa79f0
