# Dokku Kimai

Deploy [Kimai Time Tracking](https://www.kimai.org/) with Dokku.

## Docker installation

```bash
# GPG Keys
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Docker Debian Repo
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Docker installation
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test it:
docker run --rm hello-world
```

## Install and config Dokku

```bash
# Dependencies
sudo apt-get update -qq >/dev/null
sudo apt-get -qq -y --no-install-recommends install apt-transport-https

# GPG Key
wget -qO- https://packagecloud.io/dokku/dokku/gpgkey | sudo tee /etc/apt/trusted.gpg.d/dokku.asc

# Getting the OS name
DISTRO="$(awk -F= '$1=="ID" { print tolower($2) ;}' /etc/os-release)"
OS_ID="$(awk -F= '$1=="VERSION_CODENAME" { print tolower($2) ;}' /etc/os-release)"

# Dokku repo sources
echo "deb https://packagecloud.io/dokku/dokku/${DISTRO}/ ${OS_ID} main" | sudo tee /etc/apt/sources.list.d/dokku.list
sudo apt-get update -qq >/dev/null

# Install Dokku
sudo apt-get -qq -y install dokku
sudo dokku plugin:install-dependencies --core

# Config Dokku
dokku config:set --global DOKKU_RM_CONTAINER=1  # don't keep `run` containers around

# Dokku plugins
sudo dokku plugin:install https://github.com/dokku/dokku-mysql.git mysql
sudo dokku plugin:install https://github.com/dokku/dokku-maintenance.git
sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
```

Add your SSH key to Dokku, if needed

```shell
dokku ssh-keys:add admin path/to/pub_key
```

## Installing Application

Temporary and environment variables.

```shell
export ADMIN_EMAIL="mail@mydomain.me"
export APP_NAME="kimai"
export APP_DOMAIN="kimai.mydomain.me"
export MYSQL_NAME="mysql_$APP_NAME"
export ADMINMAIL="mail@mydomain.me"
export ADMINPASS="strongpass"
```

Create and config the applicatiom.

```shell
dokku apps:create $APP_NAME
dokku checks:disable $APP_NAME
dokku domains:add $APP_NAME $APP_DOMAIN
dokku letsencrypt:set $APP_NAME email $ADMIN_EMAIL

# Database service creation
dokku mysql:create $MYSQL_NAME

# Link app to database service
dokku mysql:link $MYSQL_NAME $APP_NAME

# Application environment variables
dokku config:set --no-restart $APP_NAME ADMINMAIL=$ADMINMAIL
dokku config:set --no-restart $APP_NAME ADMINPASS=$ADMINPASS

# Deploy com Git
dokku git:from-image $APP_NAME kimai/kimai2:apache
```
