FROM node:8.16-buster-slim AS build

RUN apt-get update && apt-get install -y git

RUN git clone --branch 13.0.1 --depth 1  https://github.com/nightscout/cgm-remote-monitor.git /opt/app


FROM node:8.16-buster-slim

COPY --from=build /opt/app /opt/app
WORKDIR /opt/app

RUN apt-get update && apt-get install -y python make build-essential && \
  npm install && \
  npm run postinstall && \
  npm run env

EXPOSE 1337

CMD ["node", "server.js"]
