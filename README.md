# cgm-remote-monitor-docker

## About

This is yet another docker image for Nightscout. Unlike the other images this tries to:
- use individual tags for each Nightscout release
- use Docker Hub's auto build feature
- provide minimalized images using multistage builds
- provide a `docker-compose.yml` example file
- keep it up to date


## Prerequisites

You need to have a host running [Docker](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/install/). It is recommended to use a reverse proxy supporting `https://` and `wss://` SSL offloading.

### Apache HTTPD reverse proxy

One possiblity is to use Apache HTTPD as SSL offloading reverse proxy using certificates from [Let's Encrypt](https://letsencrypt.org/).

You need to enable the *mod_proxy* and *mod_proxy_wstunnel* modules. Nightscout uses a *Websocket* connection which needs to be handled special using *mod_rewrite*:

```
<VirtualHost *:443>
  ServerName ns.example.com

  RewriteEngine On
  RewriteCond %{REQUEST_URI}  ^/socket.io            [NC]
  RewriteCond %{QUERY_STRING} transport=websocket    [NC]
  RewriteRule /(.*)           ws://127.0.0.1:1337/$1 [P,L]

  ProxyPass / http://127.0.0.1:1337/
  ProxyPassReverse / http://127.0.0.1:1337/

  SSLCertificateFile /etc/letsencrypt/live/ns.example.com/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/ns.example.com/privkey.pem
  Include /etc/letsencrypt/options-ssl-apache.conf

  # Override CORS headers to allow access by nightscout-reporter (https://nightscout-reporter.zreptil.de/)
  #<IfModule mod_headers.c>
  #  SetEnvIf Origin "https://(ns.example.com|nightscout-reporter.zreptil.de)$" AccessControlAllowOrigin=$0
  #  Header set Access-Control-Allow-Origin %{AccessControlAllowOrigin}e env=AccessControlAllowOrigin
  #  Header set Access-Control-Allow-Credentials true
  #</IfModule>
</VirtualHost>
```


## Deployment

Grab the example [docker-compose.yml](https://github.com/liske/cgm-remote-monitor-docker/blob/master/docker-compose.yml) file and change it to meet your needs. You need to use an explicit version tag for the docker image - the `latest` tag is missing intentionally!

Details of the configuration parameters can be found in Nightscout's [README.md](https://github.com/nightscout/cgm-remote-monitor/blob/master/README.md#environment).


## Upgrading

You need to check [Nightscout Releases](https://github.com/nightscout/cgm-remote-monitor/releases) to look for changes. If a docker image of `liske/cgm-remote-monitor` with a new release tag is available you just need to update your `docker-compose.yml` file. Running `docker-compose up` again will restart Nightscout using the new image.
