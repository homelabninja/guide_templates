# Nextcloud setup via docker

In this tutorial we are going to learn how to install nextcloud via docker using Caddy proxy. 

This YAML file defines a Docker Compose configuration for running a Nextcloud instance with a MariaDB database, a reverse proxy service provided by Caddy, and a shared network named nc-network.

The nc-network is defined as an external network, meaning that it was probably created before running this configuration.

The configuration also defines several named volumes that will be used to store data: nextcloud-apache2, nextcloud-data, nextcloud-config, nextcloud-db, and caddy-data.

The services section specifies three services that will be run in containers: caddy, nextcloud-db, and nextcloud-app.

The caddy service runs the lucaslorentz/caddy-docker-proxy Docker image, exposes ports 80 and 443 on the host machine, and listens for requests from the nc-network. It also mounts the /var/run/docker.sock file from the host machine and the caddy-data volume.

The nextcloud-db service runs the mariadb image and creates a MariaDB instance with the specified environment variables and volumes.

The nextcloud-app service runs the nextcloud image and connects to the MariaDB instance using the specified environment variables. It also exposes ports 1080 and 1443 on the host machine and mounts several volumes. It depends on the nextcloud-db service and includes a caddy label.

# setup

Creat a docker network

```bash
docker network create nc-network
```

create a yml file

edit the parameters according to your domain and remember to chage the default passwords


run the following command to start docker containers

```bash
docker-compose -f docker-compose.yml up -d
``` 

if the server is not running update the apache 2 ports to match 1080 and 1443.

run the docker-compose again


also point the domain to your local ip address and make sure that in your local router settings port 80 and 443 are proxied to the server running the docker container. 




## docker compose

 Docker Compose file for running a Nextcloud instance with a MariaDB database,
 a reverse proxy service provided by Caddy, and a shared network named "nc-network"


```yml
version: "3.7"

# Docker Compose file for running a Nextcloud instance with a MariaDB database,
# a reverse proxy service provided by Caddy, and a shared network named "nc-network"
######################################
# create an external network
networks:
  nc-network:
    external: true


volumes:
  nextcloud-apache2:
  nextcloud-data:
  nextcloud-config:
  nextcloud-db:
  caddy-data:


services:
  caddy:
    image: lucaslorentz/caddy-docker-proxy
    container_name: caddy_proxy
    ports:
      - 80:80
      - 443:443
    environment:
      - CADDY_INGRESS_NETWORKS=nc-network
    networks:
      - nc-network
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - caddy-data:/data
    restart: unless-stopped

  nextcloud-db:
    image: mariadb
    restart: unless-stopped
    container_name: mariadb-nxtcloud
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - nextcloud-db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=root_password
      - MYSQL_PASSWORD=sql_password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
    networks:
      - nc-network

  nextcloud-app:
    image: nextcloud
    container_name: nextcloud
    restart: unless-stopped
    volumes:
      - nextcloud-data:/var/www/html
      - nextcloud-apache2:/etc/apache2
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - nextcloud-config:/config
    environment:
      - MYSQL_PASSWORD=sql_password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=nextcloud-db
      - NEXTCLOUD_ADMIN_USER=nc_admin
      - NEXTCLOUD_ADMIN_PASSWORD=nc_admin
      - NEXTCLOUD_TRUSTED_DOMAINS=www.nextcloud.siliconmonde.com nextcloud.siliconmonde.com https://nextcloud.siliconmonde.com
      - OVERWRITEPROTOCOL=https
    ports:
      - 1080:1080
      - 1443:1443
    networks:
      - nc-network
      
    depends_on:
      - nextcloud-db
    labels:
      caddy: nextcloud.siliconmonde.com
      caddy.reverse_proxy: "{{upstreams 1080}}"  
```

