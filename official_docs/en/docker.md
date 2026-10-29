
# Virtual server with Docker

This is the currently supported Docker-based installation of OpenBioMaps.

The Docker installation can be used for **testing, development, and production environments**. The installation scripts configure the OpenBioMaps services and their dependencies in a Docker Compose environment.

## Installation overview

A new OpenBioMaps Docker installation consists of the following main steps:

1. **Prepare the server and install Docker Engine and Docker Compose**
2. **Download the OpenBioMaps Docker installation files**
3. **Configure the installation for the server**
4. **Run the OpenBioMaps installer**

The installer detects the available Docker Compose implementation and supports both the current Docker Compose plugin (`docker compose`) and the legacy standalone Compose command (`docker-compose`).

## Prepare Docker and Docker Compose

A Linux server with a supported Docker installation is required.

For a new Debian installation, the recommended method is to install Docker Engine from the official Docker APT repository. This also provides the Docker Compose plugin.

For current installation instructions, see the official Docker documentation:

[Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)

After installing Docker, verify that the Docker daemon is running:

```bash
sudo systemctl status docker
```

Test the Docker installation:

```bash
sudo docker run hello-world
```

Install the Docker Compose plugin if it was not installed together with Docker Engine:

```bash
sudo apt update
sudo apt install docker-compose-plugin
```

Check the installed Compose version:

```bash
docker compose version
```

The expected command is:

```text
docker compose version
```

not the older standalone:

```text
docker-compose --version
```

For further information about Docker Compose installation, see:

[Install the Docker Compose plugin](https://docs.docker.com/compose/install/linux/)

### Optional: use Docker without `sudo`

By default, Docker commands require root privileges. If Docker should be available to the current user without `sudo`, add the user to the `docker` group:

```bash
sudo usermod -aG docker $USER
```

Log out and log in again for the group membership to take effect.

For further information, see:

[Docker post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/)

## Start the OpenBioMaps installation

The OpenBioMaps Docker installation is started directly by the installation script. It is not necessary to clone the Docker repository manually.

### Download and run the installer

On the server where OpenBioMaps is to be installed, run:

```bash
curl -s https://gitlab.com/openbiomaps/docker/obm-composer/-/raw/master/install.sh > /tmp/install.sh && sudo bash /tmp/install.sh
```

The installer will guide you through the installation process.

### Configure the server

During the installation, the installer detects the server addresses available on the host, including:

* local IP address
* public IP address, if available
* fully qualified hostname, if available

It presents the detected addresses and proposes a suitable default server address. The administrator can accept the proposed address or enter another address appropriate for the deployment.

The installer also detects available HTTP and HTTPS ports. If the standard ports (`80` and `443`) are already in use, alternative free ports can be selected.

The generated configuration is written to the installation `.env` file. This includes the selected server address and HTTP/HTTPS ports, as well as the passwords and other values required by the Docker environment.

### Server-specific configuration

Depending on the deployment, additional configuration may be required outside the OpenBioMaps installer, for example:

* DNS configuration for the OpenBioMaps domain
* firewall rules
* SSL/TLS certificates
* SMTP/mail server configuration
* reverse proxy configuration
* external access to the selected HTTP/HTTPS ports

These settings depend on the server and network environment and are not required for every installation.

### Installation process

After the configuration has been confirmed, the installer:

1. prepares the Docker environment;
2. starts the required Docker services;
3. waits for the databases to become available;
4. initializes the OpenBioMaps databases;
5. performs the required post-installation operations;
6. installs or updates the built-in OpenBioMaps projects and other system components;
7. records the successful completion of the installation.

The installation is complete when the installer reports successful completion.

### Docker services

The `docker-compose.yml` file of the installation defines the following services:

| Service | Purpose |
|---|---|
| `app` | The OpenBioMaps web application (Apache, PHP). Publishes the HTTP and HTTPS ports selected during the installation. |
| `background-worker` | Runs the central background task worker (for example the e-mail queue) from the same application image. See [Central background task worker](server_administration.html#central-background-worker). |
| `mapserver` | MapServer for the map layers of the projects. |
| `biomaps_db` | PostgreSQL/PostGIS database server with the `biomaps` and `gisdata` databases. |
| `obm-server-api` | The OpenBioMaps server API, published on port `9001`. |
| `adminer` | Web based database manager, published on port `9882`. |
| `memcached` | Cache server used by the application. |

All services share the internal `obm_back` network.


## Visit your OBM app

[http://YOUR_SERVER_NAME/](http://YOUR_SERVER_NAME/)

[http://YOUR_SERVER_NAME/projects/sablon/](http://YOUR_SERVER_NAME/projects/sablon/)

Log in to your template database using *valaki@openbiomaps.org* user name and *abc123* password. After the first login, please change this default password!

If you installed the docker in your local computer you can access the services above in localhost.

Update your server and sablon project following this guide:

[New server installation tutorial](server_install.html)


## Database access

You can access your Postgres database on the following pre-configured online database manager applications. However, it depends on your host-docker relationship.

Adminer: [http://YOUR_SERVER_NAME:9882/](http://YOUR_SERVER_NAME:9882/)

  Adminer notes:
  
    server = biomaps_db   (the service name; the network aliases `biomaps` and `gisdata` also work)
    db_name = biomaps | gisadmin
    db_user = biomapsadmin | sablon_admin | YOUR_PROJECT_admin
    password = (check the .env file for the biomapsadmin's password or the local_vars.php.inc for ..._admin's password)

You can manage the database with the *biomapsadmin* user. This is a superuser. Its password is generated by the system during installation and can be found in the /srv/docker/openbiomaps/.env file.

For the MapServer map file, the encrypted password for the database connection can be generated with the /var/lib/openbiomaps/maps/access.key.

OpenBioMaps creates two databases by default. The 'biomaps' contains the tables needed for the system to work, while the 'gisdata' contains the data tables of the project databases. In other words, the latter is where the data collected by users will be stored and to which users can connect. The 'biomapsadmin' is a superuser in both databases. Its password is generated by the system during installation and stored in the /srv/docker/openbiomaps/.env file.

## Docker maintenance app

This step is not obligatory but can be useful if you need a strong web admin interface for docker management. 

```
mkdir -p /srv/docker/portainer && cd /srv/docker/portainer

git clone https://gitlab.com/openbiomaps/docker/obm-portainer.git .

sudo su

docker-compose pull

# Genereate a strong random password for the admin user
bash ./password_gen.sh

docker-compose up -d

On the portainer interface use the "Get started" button...

```

Visit your docker-admin (portainer) app

[http://YOUR_SERVER_NAME:9000/](http://YOUR_SERVER_NAME:9000/)

Log in to your app using *admin* user name and your password;

If you installed the docker in your local computer you can access the services above in localhost.


## OBM maintenance: Supervisor

You can access OBM server admin interface: 
[http://localhost:9880/supervisor.php](http://localhost:9880/supervisor.php)

or 

[https://yourserver.com/supervisor.php](https://yourserver.com/supervisor.php)

with *supervisor* username and password created by obm_post_install.sh. This password is located at /etc/openbiomaps/.htpasswd.

You can regenerate the Supervisor's password with `./obm_post_install.sh update supervisor` command


## Updates: update application with Docker

Run the commands in the installation directory (`/srv/docker/openbiomaps`). These commands are safe, they do not destroy the changes made through the OpenBioMaps web interface.

```console
foo@bar:~$ docker compose pull
foo@bar:~$ docker compose up -d
```

The installer supports both the Docker Compose plugin (`docker compose`) and the legacy standalone command (`docker-compose`). On older hosts replace `docker compose` with `docker-compose` in the commands of this page.

`docker compose pull` downloads the new images of all services, including `app`, `background-worker` and `obm-server-api`. `docker compose up -d` recreates the containers whose image has changed.

Update only one container:

```console
foo@bar:~$ docker compose pull app
foo@bar:~$ docker compose up -d app
```

The `background-worker` uses the same application image as `app`. After an application update recreate both services:

```console
foo@bar:~$ docker compose up -d app background-worker
```

Check the state of the services and the logs of the background worker:

```console
foo@bar:~$ docker compose ps
foo@bar:~$ docker compose logs -f background-worker
```

## Email setting for projects


You **must set up a Mail server access** to send emails from the app

Assuming that the new server does not have its domain name, the default value for sending mail is set to SMTP (/etc/openbiomaps/system_vars.php.inc), which requires you to configure outgoing smtp servers and associated authentication for each project (/var/www/html/biomaps/projects/.../local_vars.php.inc)

These config files can be edited in the supervisor interface.

Find the mail settings section and set up the SMTP host and authentication if needed.

If there is an external SMTP server here you are an example:
```console
 // Mail settings
 define('SMTP_AUTH',true); # true
 define('SMTP_HOST','mail.my-mail-server.com');
 define('SMTP_USERNAME','my-name@my-mail-server.com');
 define('SMTP_PASSWORD','something');
 define('SMTP_SECURE','tls'); # Or starttls
 define('SMTP_PORT','587'); # 465
 define('SMTP_SENDER','openbiomaps@my-mail-server.com');
```

If SMTP_SENDER is not set, the SMTP_USERNAME will be the sender. Sending mails with Google, with these simple settings is not possible, because Google uses xoauth layer to authenticate! It is possible to include that layer here!

If the host system will be the SMTP server:
```console
 // Mail settings
define('SMTP_AUTH',false);
define('SMTP_HOST','172.17.0.1');
define('SMTP_PORT','25');
define('SMTP_SENDER','info@you-smtp-server');
```
For the IP address above check host "ip addr | grep docker0"

On the host depending on what MTA you have here you some examples:

### Exim4

In the /etc/exim4/update-exim4.conf file

 dc_relay_nets='172.21.0.0/16'
 
 dc_local_interfaces='127.0.0.1 ; ::1 ; 172.17.0.1' 

these lines should be updated, but depending on your exim config maybe something else as well.

In the /etc/exim4/exim4.config file

"hostlist   relay_from_hosts..." line, you should extend with the obm_back network e.g.

  hostlist   relay_from_hosts = localhost :172.20.0.0/16 :172.17.0.0/16 :172.21.0.0/16
 
  Comment: "Maybe one of the three networks is enough above, I did not test yet"

### Postfix

inet_interfaces = 172.17.0.1

mynetworks = 172.21.0.4 172.20.0.6

Here's how to find out docker networks and ip addresses: 

```console
docker container ls
```
Search for the `app` container (its name ends with `app_1` or `app-1`, depending on the Docker Compose version)

```console
docker inspect xxxxx_obm-composer_app_1
```
Search for obm_back and obm_web interfaces:

  obm-composer_obm_back {
  
  ...
  
  "IPAddress": "172.20.0.6",
  
  }
  
  obm-composer_obm_web {
  
  ...
  
  "IPAddress": "172.21.0.4",
  
  }

### Firewall

You may also need to update your firewall to enable incoming mail from the image to the host. The network address of obm_back must be allowed as an incoming network for the firewall. E.g.
```console
ufw allow from 172.20.0.0/16 proto tcp to any port 25
```
### Global SMTP settings

Most probably you want to use the same SMTP settings for all projects on your server. In this case, use the

   - SMTP_GLOBAL_HOST
   - SMTP_GLOBAL_AUTH
   - SMTP_GLOBAL_USERNAME
   - SMTP_GLOBAL_PASSWORD
   - SMTP_GLOBAL_SECURE
   - SMTP_GLOBAL_PORT
   - SMTP_GLOBAL_SENDER
 
  parameters in the sytem_vars.php.inc. At least the SMTP_GLOBAL_HOST should be set if you want to use global parameters. The local parameters override the globals always.
 

## Setting up **ssl**/**https access** (highly recommended)


You may need to update your project access protocol setting in the Supervisor however it depends on your host's setting.

*There is no webserver on the Host, but Host provide ssl certs for docker*

One possible way is to use the host's SSL certificates by the way to mount the necessary directories from the host to the docker.
You can create letsencrypt 
``` console
apt install dehydrated
vi /etc/dehydrated/domain.txt
    YOURDOMAIN
dehydrated -c
```

docker-compose.yml:
```console
services:
  app:
    image: registry.gitlab.com/openbiomaps/web-app:latest
    volumes:
      ...
      - /etc/letsencrypt/YOURDOMAIN:/etc/apache2/certs
      - ./apache2/default-ssl.conf:/etc/apache2/sites-enabled/default-ssl.conf
    ports:
     - 80:80
     - 443:443
    ... 
```

*Host has a web server and provides a proxy for the docker*

Another way to use the host's Apache proxy

Host: /etc/apache2/sites-enabled/000-default.conf
```
RedirectMatch permanent ^(?!/.well-known/.*) https://YOURDOMAIN/
```

Host: /etc/apache2/sites-enabled/default-ssl.conf
```
RequestHeader set X-Forwarded-Proto 'https'
RequestHeader set X-Forwarded-Host 'YOURDOMAIN'
RequestHeader set X-Forwarded-Port "443"

ProxyPass /.well-known !
ProxyPass / 
http://localhost:8090/

ProxyPassReverse / 
http://localhost:8090/

ProxyPreserveHost On
<Proxy *>
allow from all
</Proxy> 
```

docker-compose.yml:
```console
services:
  app:
  ...
      ports:
      - 80:8090
  ...
```
In this case, you don't need to use HTTPS protocol in project settings because the OBM can recognize the HTTPS request through the HTTP-X-FORWARD settings.


*Using traefik to process different domain request in the docker level. E.g. you have several docker containers on your host...*

To set up docker based https trafic rooter we recommend using traefik2.x in another container:

https://gitlab.com/openbiomaps/docker/traefik2.0-proxy

And update your docker-compose.yml file to communicate with traefik:

```console
networks:
   traefik20_default:
    external: true 
  #obm_web:
  #  external: true

services:
  app:
  ....
# Do not use ports, traefik provides them!!!
#    ports:
#      - 80:80
#      - 443:443
    networks:
      - obm_back
      #- obm_web
      - traefik20_default
    labels:
      - traefik.enable=true
      - traefik.docker.network=traefik20_default
      - traefik.http.routers.obm-secured.rule=Host(`YOUR_DOMAIN`)
      - traefik.http.routers.obm-secured.entrypoints=https
      - traefik.http.routers.obm-secured.middlewares=hsts@file
      - traefik.http.routers.obm-secured.tls.certresolver=letsencrypt
      - "traefik.http.middlewares.obm-biotika-redirect.redirectregex.regex=^https?://biotika.YOURDOMAIN(.*)"
      - "traefik.http.middlewares.obm-biotika-redirect.redirectregex.replacement=https://YOUR_DOMAIN/projects/YOURPROJECT/"
      - traefik.http.middlewares.obm-biotika-redirect.redirectregex.permanent=true
  
  phppgadmin:
  ...
    networks:
        #- obm_web
      - obm_back
      - traefik20_default
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik20_default"
      - "traefik.http.routers.obm-pgadmin.rule=Host(`phppgadmin.YOURDOMAIN`)"
      - "traefik.http.routers.obm-pgadmin.entrypoints=https"
      - "traefik.http.routers.obm-pgadmin.tls.certresolver=letsencrypt"
      - "traefik.http.services.obm-pgadmin.loadbalancer.server.port=8080"
volumes:
  ...
    traefik20_letsencrypt:
    external: true
```

This latter example is maybe not complete yet...

If you provide Postgres access you also need to set up SSL over Postgres

If you have Traefik you can configure SSL access there. In another case, you can give SSL certs for your database container and set up Postgres to accept connection only through SSL.

docker-compose.yml:
```console
services:
...
  biomaps_db:
     volumes:
       - /PATH_TO_CERTS/ssl.cert:/etc/ssl/certs/YOURDOMAIN.cert
       - /PATH_TO_CERTS/ssl.key:/etc/ssl/certs/YOURDOMAIN.key
       - ./postgresql.conf:/var/lib/postgresql/data/postgresql.conf 
       - ./pg_hba.conf:/var/lib/postgresql/data/pg_hba.conf 
```

In biomaps_db container:
/..../pg_hba.conf:
```console
hostssl all all all md5 
```

/..../postgres.conf:
```console
ssl = on
ssl_cert_file = '/etc/ssl/certs/YOURDOMAIN.cert'
ssl_key_file = '/etc/ssl/certs/YOURDOMAIN.key' 
```
You can try your postgres connection without ssl:
```console
psql "postgresql://gisadmin@YOURDMAIN:5432/gisdata?sslmode=disable"
```

If your SSL require works, you will get an error message like this:

psql: FATAL:  no pg_hba.conf entry for host "xxxxxxx", user "gisadmin", database "gisdata", SSL off 


## Docker maintenance

### Stopping docker

```console
foo@bar:~$ docker compose down
```

### Drop everything (including data and databases)

```console
foo@bar:~$ docker compose down -v
```


### Shell access of the system in the container image

```console
foo@bar:~$ docker compose exec app bash
```
Here we accessed the **app** service. See service names in the docker-compose.yml file.


### Reading logs

```console
foo@bar:~$ docker compose logs -f app
```

### Using pgtop

docker compose exec -u postgres <service_name> pg_top

### Restart app

Do not restart Apache from the Docker shell, but from outside
```console
foo@bar:~$ docker compose restart app
```

### Remove huge amounts of old, not used docker images

Do we have a lot?
```console
docker images | grep "<none>"
```
Let's drop them...
```console
docker images | grep "<none>" | awk '{print $3}' | sed -e 's/^/docker rmi /' | bash
```
You may need to edit the traefik2.0/traefik.yml, traefik2.0/docker-compose.yml, and traefik2.0/acme.json files

### Auto update of docker

https://github.com/OpenBioMaps/scripts/blob/master/docker-auto-update.readme

## Archive tables, data, ...

For scripts follow this repo:

https://github.com/OpenBioMaps/scripts

- use this to set up a periodic sql dumps of important tables: archive.sh

### crontab setting examples for archive.sh

```sh
   #dumping normal tables from Monday to Saturday
   15 04 * * 1-6 /root/archive.sh normal &
   #dumping all tables and whole databases on every Sunday
   15 04 * * 7 /root/archive.sh full &
```

### Example settings in obm_archive_settings.sh

```sh
  #path of table list
  table_list="${HOME}/.archive_list.txt"

  #postgres parameters
  project_database="gisdata"
  system_database="biomaps"
  admin_user="gisadmin"
  archive_path="/home/archives"
  pgport="5432"
  pg_dump="pg_dump -p $pgport"
  psql="psql -p $pgport"

  #FOR DOCKER based OBM systems
  #docker="/usr/bin/docker-compose -f /path/to/docker-compose.yml exec -T"
  #pg_dump="$docker biomaps_db pg_dump -p $pgport"
  #psql="$docker biomaps_db psql -p $pgport"
  
  #table dayof_week dayofmonth month
  #foo at every day
  #foo * * *
  #bar every Monday
  #bar 1 * *
  #casbla at every 1st day of every June
  #casbla * 1 6
```

### Dumping table from the database using docker

docker-compose exec -T biomaps_db bash -c "pg_dump -U biomapsadmin --table public.YOUR_TABLE gisdata" > YOUR_TABLE.sql



## Resources


* https://gitlab.com/openbiomaps/web-app
* https://gitlab.com/openbiomaps/docker/obm-composer
