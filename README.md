# Dokku WordPress

A simple guide to deploy the [Bitnami WordPress docker image](https://github.com/bitnami/containers/tree/main/bitnami/wordpress) on Dokku.

## Installation

### Create a new app on your Dokku server

```bash
dokku apps:create my-wordpress
```

### Create a new database for your app

```bash
dokku mariadb:create my-wordpress-db
```

### Link the database to your app

```bash
dokku mariadb:link my-wordpress-db my-wordpress
```

### Create a new storage for your app

```bash
dokku storage:ensure-directory my-wordpress --chown=false

sudo chown -R 1001:1001 /var/lib/dokku/data/storage/my-wordpress
```

### Mount the storage to your app

```bash
dokku storage:mount my-wordpress /var/lib/dokku/data/storage/my-wordpress:/bitnami/wordpress
```

### Set environment variables for your app

Replace the values with the results from `mariadb:link`

```bash
# Should look like this
#mysql://<user>:<password>@<host>:<port>/<database>
mysql://mariadb:c89bf4a35681cb17@dokku-mariadb-my-wordpress-db:3306/my_wordpress_db
```

```bash
dokku config:set my-wordpress \
  WORDPRESS_DATABASE_HOST=REPLACE_ME \
  WORDPRESS_DATABASE_NAME=REPLACE_ME \
  WORDPRESS_DATABASE_USER=REPLACE_ME \
  WORDPRESS_DATABASE_PASSWORD=REPLACE_ME \

  WORDPRESS_ENABLE_HTACCESS_PERSISTENCE=yes \
  WORDPRESS_PLUGINS=none \

  WORDPRESS_ENABLE_REVERSE_PROXY=yes \

  PHP_EXPOSE_PHP=Off \
  PHP_MEMORY_LIMIT=256M \
  PHP_POST_MAX_SIZE=512M \
  PHP_UPLOAD_MAX_FILESIZE=512M \
```

[All available environment variables](https://github.com/bitnami/containers/tree/main/bitnami/wordpress#environment-variables)

### Port mapping

```bash
dokku proxy:set my-wordpress nginx

dokku proxy:ports-add my-wordpress http:80:8080 https:443:8080

dokku proxy:build-config my-wordpress
```

### Deploy your app

```bash
dokku git:from-image my-wordpress bitnami/wordpress:6.1.1
```

### Configure Dokku Nginx (optional)

First create the directory

```bash
sudo mkdir /home/dokku/my-wordpress/nginx.conf.d/

echo "client_max_body_size 512M;" | sudo tee /home/dokku/my-wordpress/nginx.conf.d/99-nginx.conf

sudo chown -R dokku:dokku /home/dokku/my-wordpress/nginx.conf.d/

dokku proxy:build-config my-wordpress
```

### Set up Object Cache (optional)

```bash
dokku redis:create my-wordpress-redis
dokku redis:link my-wordpress-redis my-wordpress

# Should look like this
# redis://:<password>@<host>:<port>
redis://:2b876b700dd27cefed31472985d9bb80bef49b5f71fded8361bcc2a8d6bb7990@dokku-redis-my-wordpress-redis:6379
```

```bash
dokku config:set my-wordpress WORDPRESS_EXTRA_WP_CONFIG_CONTENT="define('WP_REDIS_HOST', 'REPLACE_ME'); define( 'WP_REDIS_PASSWORD', 'REPLACE_ME' );"
```

And then install the [Redis Object Cache](https://wordpress.org/plugins/redis-cache/) plugin.
