FROM nginx:latest
LABEL maintainer=jodyvole
WORKDIR /home/
COPY ./nginx.conf /etc/nginx/nginx.conf
COPY ./mini_web_server.c .
COPY ./run.sh .
RUN apt-get update && apt-get upgrade -y && apt-get install -y gcc libfcgi-dev spawn-fcgi
ENTRYPOINT [ "sh", "./run.sh" ]