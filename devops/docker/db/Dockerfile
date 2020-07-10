FROM mysql:5.7

EXPOSE 3306

COPY . /code/devops/docker/db

RUN set -eux \
 && cd /code/devops/docker/db \
 && cp __etc__mysql__conf.d__my.conf /etc/mysql/conf.d/my.cnf

CMD ["mysqld"]
