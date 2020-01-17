# Generic Reverse Proxy with HTTPS for Docker

Template for a generic docker-compose nginx reverse proxy with Let's Encrypt companion and an example of how to connect other composes to the reverse proxy.

## Requirements
* docker-compose.yml in version 3.5 (or higher)
* docker-compose version 1.18.0 (or higher)
* Docker engine 17.06.0 (or higher)

## Usage

```bash
wget https://raw.githubusercontent.com/stigvoss/docker-reverse-proxy/master/docker-compose.yml
docker-compose up -d
```

### Connecting a compose to the reverse proxy.

Below is an example of connecting a Roundcube container for exposed as mail.example.com through the reverse proxy.

```dockerfile
version: '3.5'

services:
  roundcube:
    container_name: roundcube
    image: roundcube/roundcubemail
    environment:
     - VIRTUAL_HOST=mail.example.com
     - LETSENCRYPT_HOST=mail.example.com
     - LETSENCRYPT_EMAIL=postmaster@example.com
    volumes:
     - /srv/roundcube/html:/var/www/html
    networks:
     - reverse-proxy

networks:
  reverse-proxy:
    external:
      name: reverse-proxy
```

The trick is to use the network created by the reverse proxy compose, which is achieved by defining a network in the compose as external and either providing the network name using the name attribute (shown above) or naming the network the same as the network created by the reverse proxy compose (see below).

```dockerfile
networks:
  reverse-proxy:
    external: true
```

This allows the reverse proxy to discover and adopt your container.

## Note: Requirement for docker-compose version 3.5

It is not strictly necessary to have docker-compose version 3.5 for achieving what is done in this docker-compose, but this exact compose uses named networks (see the example below) to make it easier to ensure the reverse proxy network name is predictable.

```dockerfile
networks:
  proxy:
    name: reverse-proxy
```

By removing the `name: reverse-proxy`, version 3.5 is not required, but the network will be named according to the compose name and you have to make sure to alter the external network name accordingly.

