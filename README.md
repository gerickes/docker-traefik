# docker-traefik
Use Traefik as reverse proxy as Docker container.

## Description

This repository will describe you quickly how to setup *Traefik* as reverse proxy as a Docker container on your system.

## Requirement

### Docker

Please create your Docker installation. As guide to do your installation on your server youo can take [Docker einrichten unter Linux, Windows, macOS](https://www.heise.de/ct/artikel/Docker-einrichten-unter-Linux-Windows-macOS-4309355.html) provided by *c't*.

### Network

Traefik will use the network `web` for the traffic between the different Docker conainters. Please create this network with:
``` bash
docker network create web
```

## Files

### docker-compose.yml

This file witl use the official image provided by Traefik in the latest version. At the moment it is version 2.5. We will use `web` for our network and using port *80* and *443*. In the folder `letsencrypt` we will store the `acme.json` file for the certificats we are getting from Let´s Encrypt.

### traefik.yml

This is the configuratriopn file for Traefik. All connections coming from port *80* will be redirected directly to *443*. The *API Dashboard* will be enabled. The provider for our certificates will be *Let´s encrypt*. Here you have to update the field `email`. With file `traefik_api.yml` we are providing the configuration file for the *API Dashboard* website.

### traefik_api.yml

With this file you will do the setting of your *API Dashboard*. To secure this site you will use *HTTP Basic Authentication*. Please create your secret with the following command:
``` bash
docker run --rm httpd htpasswd -nb admin <your secret>
```
Enter this secret in your configuraton file. Please also update the host to get access to you *API Dashboard*.

## Your final web service

Please add the following lines in your final Docker container in the `docker-compose.yaml`-file:

``` YAML
# file: docker-compose.yml
version: "3"
services:
  <your service>:
#(...)
    labels:
      - traefik.enable=true
      - traefik.http.routers.recipes.rule=Host(`<your domain>`)
      - traefik.http.routers.recipes.tls=true
      - traefik.http.routers.recipes.tls.certresolver=lets_encrypt"
      - traefik.http.services.recipes.loadbalancer.server.port=80
#(...)
    networks:
      - web

networks:
  web:
    external: true
#(...)
```

The `Host` will be the domain of your web service and *Let´s Encrypt* is the provider for the certificate. I was able to get with this configuration [Vaultwarden](https://github.com/dani-garcia/vaultwarden) and [Tandoor Recipes](https://docs.tandoor.dev/) running behind *Traefik* as reverse proxy.

## Reference

Reference for this documentation was the book writen by **Bernd Öggl and Michael Kofler**, Docker - Das Praxisbuch für Entwickler und DevOps-Teams, cahpter *9.6 Traefik Proxy*.
