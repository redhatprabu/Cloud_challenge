FROM node

LABEL maintainer="pinehead@linuxacademy.com"
LABEL org.label-schema.applicaiton=weather-app
LABEL org.label-schema.version=v1.1

RUN mkdir -p /var/node
ADD src/ /var/node/
WORKDIR /var/node
RUN npm install
EXPOSE 3000
CMD ./bin/www
