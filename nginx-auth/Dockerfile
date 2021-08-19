FROM ubuntu:latest

ARG USER1_PASS=local
ARG USER2_PASS=local
ARG USER3_PASS=local

ENV USER1_PASS ${USER1_PASS}
ENV USER2_PASS ${USER2_PASS}
ENV USER3_PASS ${USER3_PASS}

RUN export DEBIAN_FRONTEND=noninteractive
RUN ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime

RUN apt-get update && apt-get install -y tzdata nginx vim apache2-utils
RUN mkdir /etc/apache2/ && htpasswd -cb /etc/apache2/.htpasswd user1 ${USER1_PASS} && htpasswd -b /etc/apache2/.htpasswd user2 ${USER2_PASS} && htpasswd -b /etc/apache2/.htpasswd user3 ${USER2_PASS}

COPY ./demo-index.html /usr/share/nginx/html/demo-index.html
COPY ./app.conf /etc/nginx/conf.d/default.conf

RUN rm -rf /etc/nginx/sites-enabled/default

CMD ["nginx", "-g", "daemon off;"]