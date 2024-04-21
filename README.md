# Pterodactyl Panel & Wings in Docker

# 📚 About
There’s a lack of information about setting up and running Pterodactyl Panel inside docker using Traefik as a reverse proxy. This guide focuses on the fastest and easiest way to do that! This setup has been tested and is currently running in a production environment. All of images used in this setup are from official sources and even uses official compose files provided by Pterodactyl with addition of Traefik.

# 📦 About this Fork
This fork just updates some things, treat this as unfinished/not feature complete.
You are probably better off using the official [docker compose example](https://github.com/pterodactyl/panel/blob/develop/docker compose.example.yml) and modifying it to your needs.

Original repo: [BeefBytes/pterodactyl-docker](https://github.com/EdyTheCow/pterodactyl-docker)

# 🧰 Getting Started
This guide assumes you have at least two servers, one for panel and one for wings. You're fine by using a cheap VPS for the panel, while wings may require a higher spec server depending on the game servers you're planning to run.

### Requirements
- Domain
- Two or more servers
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose](https://docs.docker.com/compose/install/)

### DNS
If you just want to use localhost for testing purposes, you can skip this step.
Create `A record` pointing to panel's server IP, if you're using Cloudflare you can proxy this record through Cloudflare.

Create second `A record` pointing to wings server IP. If you're using Cloudflare, do not proxy this record! There's no advantages for proxying wings server. If it is proxied the server SFTP details in the panel will point to Cloudflare's IP rather than wings.

# 🏗️ Installation

### Preparations / Setting up Traefik
**Clone repository**
```
git clone https//github.com/frkri/pterodactyl-docker.git
```

**Set correct acme.json permissions**

Navigate to `_base/compose` and run
```
sudo chmod 600 acme.json
```

**Start docker compose**
Inside of `_base/compose` run
 ```
docker compose up -d
 ```

### Setting up Panel

**Configure variables**
Navigate to `panel/compose/.env` and set these variables

| Variable            |  Example  | Description                                                                                                |
|---------------------|:---------:|------------------------------------------------------------------------------------------------------------|
| PANEL_DOMAIN        | localhost | Use the domain you pointed to panel's server earlier                                                       |
| MYSQL_ROOT_PASSWORD |     -     | Use a password generator to create a strong password                                                       |
| MYSQL_PASSWORD      |     -     | Don't reuse your root's password for this, generate a new one                                              |
| APP_KEY             |  base64:  | Generate a new key using `php artisan key:generate --show` or `echo -n 'base64:'; openssl rand -base64 32` |

If using localhost as `PANEL_DOMAIN` make sure to comment out the HTTPS Traefik labels in `docker compose.yml` file under the panel service.
Also set APP_URL to `http://localhost` if you're using localhost as `PANEL_DOMAIN`.

Rest of the variables can be set as desired, these 4 are required for panel's basic functionality.

**Start docker compose**
Inside of `panel/compose` run
 ```
docker network create pterodactyl
docker volume create pterodactyl_database
docker compose up -d
 ```
Navigate to the domain you've set for `PANEL_DOMAIN` earlier and make sure the panel is up and running.

**Create a new user**
Inside of `panel/compose` run
 ```
docker compose exec panel php artisan p:user:make
 ```
Login into the panel using newly created user.

**Create a new node**
Navigate to the admin control panel and add a new `Location`. Then navigate to `Nodes` and create a node.

| Setting      |    Set to    | Description                                              |
|--------------|:------------:|----------------------------------------------------------|
| FQDN         | Wings domain | Domain you pointed to wings server, must match in `.env` |
| Behind Proxy | Behind Proxy | Set this to `Behind Proxy` for Traefik to work properly  |

You can leave `Daemon Server File Directory` as is unless you want to store server data in a specific location.
In that case make sure to read instruction in `wings/compose/docker-compose.yml`.

If using localhost as `FQDN` make sure to comment out the HTTPS Traefik labels in `docker compose.yml` file under the wings service.

The Rest of the settings can be set as you desire.
Otherwise, proceed with the guide.

### Setting up Wings
The guide assumes you're setting this up on a different server than the panel is running on!
Go back to the `Preparations / Setting up Traefik` section and follow the same instruction for setting up Traefik.

**Configure variables**
Navigate to `wings/compose/.env` and set `WINGS_DOMAIN` to the domain you pointed to wings server earlier.

**Copying daemon's config**
Navigate to the panel in your web browser and find the node you created earlier. Click on `Configuration` tab and copy the contents into `wings/data/wings/etc/config.yml`.

**Start docker compose**
 ```
docker volume create pterodactyl_wings
docker compose up -d
 ```
