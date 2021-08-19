FROM ubuntu:latest

RUN export DEBIAN_FRONTEND=noninteractive
RUN ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime

RUN apt-get update && apt-get install -y tzdata nginx vim 

COPY ./demo-index.html /usr/share/nginx/html/demo-index.html
COPY ./app.conf /etc/nginx/conf.d/default.conf

RUN rm -rf /etc/nginx/sites-enabled/default

CMD ["nginx", "-g", "daemon off;"]