# Dokku WordPress

A simple guide to deploy the [official WordPress docker image](https://hub.docker.com/_/wordpress) on Dokku.

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
dokku storage:ensure-directory my-wordpress

# Create WordPress content directory
sudo mkdir /var/lib/dokku/data/storage/my-wordpress/wp-content
sudo chown -R dokku:dokku /var/lib/dokku/data/storage/my-wordpress/wp-content
```

### Mount the storage to your app

```bash
dokku storage:mount my-wordpress /var/lib/dokku/data/storage/my-wordpress/wp-content:/var/www/html/wp-content
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
  WORDPRESS_DB_HOST=REPLACE_ME \
  WORDPRESS_DB_USER=REPLACE_ME \
  WORDPRESS_DB_PASSWORD=REPLACE_ME \
  WORDPRESS_DB_NAME=MY-WORDPRESS-DB \

  # The following are optional

  WORDPRESS_DEBUG=false \
  WORDPRESS_CONFIG_EXTRA= \
```

### Port mapping

```bash
# dokku proxy:set my-wordpress nginx

dokku proxy:ports-add my-wordpress http:80:80 https:443:80

dokku proxy:build-config my-wordpress
```

### Deploy your app

```bash
dokku git:from-image my-wordpress wordpress:6.1.1-php8.1
```

### Configure WordPress (optional)

```bash
dokku config:set my-wordpress WORDPRESS_CONFIG_EXTRA="
/* Performance */
define('WP_CACHE', true);
define('COMPRESS_CSS', true);
define('COMPRESS_SCRIPTS', true);
define('CONCATENATE_SCRIPTS', false);
define('ENFORCE_GZIP', true);
/* WP */
define('WP_POST_REVISIONS', false);"
```

### Configure PHP (optional)

First create the directory

```bash
sudo mkdir /var/lib/dokku/data/storage/my-wordpress/config
sudo chown -R dokku:dokku /var/lib/dokku/data/storage/my-wordpress/config
```

Create a file named `php-config.ini` with the following content:

`sudo nano /var/lib/dokku/data/storage/my-wordpress/config/php-config.ini`

```ini
file_uploads = On
upload_max_filesize = 512M
post_max_size = 512M

; Disable X-Powered-By
expose_php = Off

; Optional
memory_limit = 256M
max_execution_time = 300
```

Mount the file to your app

```bash
dokku storage:mount my-wordpress /var/lib/dokku/data/storage/my-wordpress/config/php-config.ini:/usr/local/etc/php/conf.d/99-php-config.ini
```

### Configure Nginx (optional)

First create the directory

```bash
# sudo mkdir /home/dokku/my-wordpress/nginx.conf.d/

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
dokku config:set my-wordpress WORDPRESS_CONFIG_EXTRA="define('WP_REDIS_HOST', 'REPLACE_ME'); define( 'WP_REDIS_PASSWORD', 'REPLACE_ME');"
```

And then install the [Redis Object Cache](https://wordpress.org/plugins/redis-cache/) plugin.
