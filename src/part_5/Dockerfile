FROM nginx:1.27.2
LABEL maintainer=jodyvole

COPY ./nginx.conf /etc/nginx/nginx.conf
COPY ./mini_web_server.c .
COPY ./my-entrypoint.sh /docker-entrypoint.sh

USER root

RUN apt-get update && apt-get install -y tcc libfcgi-dev spawn-fcgi \
    && tcc -o mini_server mini_web_server.c -lfcgi \
    && rm -rf /var/lib/apt/lists/*

RUN chown nginx:nginx /mini_server \
    && mkdir /var/cache/nginx/client_temp \
    && touch /var/log/nginx/error_log \
    && chown -R nginx:nginx /etc/nginx/nginx.conf \
    && chown -R nginx:nginx /var/cache/nginx \
    && chown -R nginx:nginx /var/log/nginx/error_log \
    && touch /var/run/nginx.pid \
    && chown -R nginx:nginx /var/run/nginx.pid \
    && chmod 777 /docker-entrypoint.sh

USER nginx

HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:8080/ || exit 1