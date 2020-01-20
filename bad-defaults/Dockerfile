FROM bitnami/node:latest
MAINTAINER "Liran Tal"
ARG NPM_TOKEN

RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y imagemagick
RUN mkdir ~/project
ADD app/. ~/project
WORKDIR ~/project
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc
RUN npm install
CMD node index.js
