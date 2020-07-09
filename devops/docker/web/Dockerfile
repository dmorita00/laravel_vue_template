FROM nginx:1.16

ARG PROJECT
ARG DEPLOY

RUN set -eux \
 && apt-get update \
 && apt-get install -y curl

COPY devops/docker/web /code/devops/docker/web

RUN set -eux \
 && cd /code/devops/docker/web \
 && mv /etc/nginx/conf.d/default.conf /etc/nginx/_default.conf \
 && cp __etc__nginx__nginx.conf /etc/nginx/nginx.conf \
 && cp __etc__nginx__conf.d__${DEPLOY}.conf /etc/nginx/conf.d/${DEPLOY}.conf
