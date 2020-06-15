# Production Docker deployment for Syspass

This project enables a Docker-based deployment of [Syspass](https://www.syspass.org/en), based on examples provided in [the Syspass documentation](https://syspass-doc.readthedocs.io/en/3.1/installing/docker.html).

## Notes
- Deploys an instance of Syspass 3.1.2, using several containers with `docker-compose`:
  - Syspass
  - MariaDB
  - Nginx
- SSL certificates are automatically generated using [docker-letsencrypt-nginx-proxy-companion](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion)

## Requirements
- A Docker instance running version `18.06.0+` of the Docker engine that supports version `3.7` or greater of the `docker-compose` file specification.
- An environment where port `80` and `443` of the host machine can be forwarded to the Nginx container for Syspass (`80` is required for Let's Encrypt certificate generation.)

## Instructions
Create the following files that contain configuration information not already defined in `docker-compose`:
- `./.env` should contain definitions for `SUBNET` and `SYSPASS_IP`. `SUBNET` is the network specification for the default network that `docker-compose` will use for Syspass (e.g. `172.30.0.0/24`), and `SYSPASS_IP` is an IPv4 address _within that subnet_ that `docker-compose` should assign to the `syspass` container (e.g. `172.30.0.10` if using the earlier value for `SUBNET`). This is _required_ to avoid an issue where Syspass can no longer authenticate against the database that was initialized if the IP address of the Syspass container ever changes (e.g. due to running `docker-compose down` and `docker-compose up` again). This issue is discussed but unresolved [here](https://github.com/nuxsmin/sysPass/issues/1299).
- `./mariadb/mariadb-variables.env` should contain a definition for `MYSQL_ROOT_PASSWORD` with the root password of the MariaDB instance. You will need this password during Syspass setup - choose something secure.
- `./syspass/syspass-variables.env` should contain definitions for:
  - `VIRTUAL_HOST` and `LETSENCRYPT_HOST` which are the publicly accessible hostname of your Syspass instance, e.g. `syspass.yourdomainhere.com`.
  - `DEFAULT_EMAIL`, an email address that the Let's Encrypt project can use to email you about important information related to your SSL certificates (e.g. expiry warnings, security issues).

Once those files are configured, you should be able to start Syspass with `docker-compose -d up`. Please note that Syspass may take a minute or two the first time it is started.
