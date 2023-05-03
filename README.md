# Introduction
[Filestash](https://github.com/mickael-kerjean/filestash) with [NGINX](https://github.com/nginx) as reverse proxy.

The docker compose file defines a stack with two services Filestash and NGINX reverse proxy using the official Docker images.

# Requirements
A Linux server with [docker](https://docs.docker.com/install/) and [docker-compose](https://docs.docker.com/compose/install/) installed is required.

# Initial configuration
1. `cd` into a directory where `docker-compose.yml` and other related files will reside, e.g. `cd /srv/docker/filestash` and copy the files from this repository into it.
2. Create the directory for SSL certificate, its private key and DH parameters: `mkdir -p ./nginx-data/certs`.
3. Generate self-signed key and certificate pair: `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./nginx-data/certs/nginx-selfsigned.key -out ./nginx-data/certs/nginx-selfsigned.crt`.

   You will be asked a series of questions. The most important line is the one that requests the `Common Name (e.g. server FQDN or YOUR name)`. You need to enter the domain name associated with your server or your server's IP address.  
   _NB:_ If you are going to add a self signed certificate to the trusted certificates on your device, you will have to modify `openssl.conf` to include your domain into `subjectAltName` when creating certificate, otherwise, browsers will not trust it. See [this post](https://stackoverflow.com/a/21494483/108686) for more details.
4. Generate DH params: `sudo openssl dhparam -out ./nginx-data/certs/dhparam.pem 2048`
5. Set the application domain or server IP address to `APP_HOST` in `.env` file. Optionally change the port `APP_PORT`.
6. Pull the Docker images: `docker compose pull`
7. According to the [Filestash installation guide](https://www.filestash.app/docs/install-and-upgrade/#optional-using-a-bind-mount-for-persistent-configuration) Filestash currently does not create the needed files in an empty mount. Instead, you need to copy the base configuration after the first start.

   * Create the directory for Filestash data: `mkdir -p ./fs-data`
   * Comment out the volumes section in `docker-compose.yml`:
      ```
      #volumes:
      # - ./fs-data/state:/app/data/state
      ```
   * Start the container: `docker-compose up -d`
   * Copy the app data from container to the mount directory: `docker cp filestash:/app/data/state ./fs-data/`
   * Stop the container: `docker-compose down`
   * Uncomment the volumes section in `docker-compose.yml`:
      ```
      volumes:
       - ./fs-data/state:/app/data/state
      ```
8. Start the container: `docker-compose up -d`.
9. Navigate to `https://<your domain>:8443`. You will be asked to set an admin password.

# Other details
By default all assets files like js, css, images, etc are set to expire in 1 month in browser cache. Content files like images and their thumbnails are set to expire in 10 days. This can be changed in `./nginx-data/templates/configs/cache.conf`.