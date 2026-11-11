FROM httpd:2.4-alpine

RUN apk add --no-cache curl

COPY certs/server.crt /usr/local/apache2/conf/server.crt
COPY certs/server.key /usr/local/apache2/conf/server.key

RUN sed -i \
    -e 's/^#\(LoadModule ssl_module\)/\1/' \
    -e 's/^#\(LoadModule socache_shmcb_module\)/\1/' \
    -e 's/^#\(Include conf\/extra\/httpd-ssl.conf\)/\1/' \
    /usr/local/apache2/conf/httpd.conf \
 && echo "ServerName localhost" >> /usr/local/apache2/conf/httpd.conf

COPY index.html /usr/local/apache2/htdocs/

EXPOSE 443

HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD curl -fk https://localhost/ || exit 1