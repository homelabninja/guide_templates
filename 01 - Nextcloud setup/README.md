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

