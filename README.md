FROM registry.gear.ge.com/predixmachine/openjdk-jre-x86_64:8u131

#switch to our app directory

ENV http_proxy=http://proxy.jfwtc.ge.com:8080
ENV https_proxy=http://proxy.jfwtc.ge.com:8080

RUN apk update && apk add json-c bison && apk add build-base

RUN apk add --no-cache bash

RUN apk add wget

RUN mkdir /var/app/

COPY json/* /usr/include/json/

WORKDIR /var/app/

COPY *.c /var/app/

COPY *.h /var/app/

COPY NVRAM.txt .

COPY data/*.csv ./data/

RUN gcc -fPIC -c *.c
RUN gcc -shared -o eco_adaptive.so *.o -lm /usr/lib/libjson-c.so.2
RUN mkdir /app
RUN mkdir /app/algoconfig
RUN mkdir /app/appconfig
RUN mkdir /app/applib
RUN mkdir /app/application-logs


WORKDIR /app/

RUN cp /var/app/*.so /app/applib/

COPY appconfig/*.json /app/appconfig/

ADD ./ren-appInfra-0.0.1-SNAPSHOT.jar /app/ren-appInfra-0.0.1-SNAPSHOT.jar

COPY start_eco.sh .

CMD ["/bin/sh", "/start_eco.sh"]


















FROM node:8.11.4 as node
WORKDIR /app

COPY . .
RUN npm i yarn

RUN yarn global add @angular/cli@latest
RUN ng build --configuration=staging
RUN ng build –prod
FROM nginx:alpine
COPY --from=node /app/dist/angular-docker-deployment usr/var/apache-tomcat-9.0.17/
COPY --from=node /app/.docker/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=node /app/dist usr/var/apache-tomcat-9.0.17/
