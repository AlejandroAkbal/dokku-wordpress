# Dokku WordPress

A simple guide to deploy WordPress on Dokku.

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
sudo mkdir /var/lib/dokku/data/storage/amiwis/wp-content
sudo chown -R dokku:dokku /var/lib/dokku/data/storage/amiwis/wp-content
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
  WORDPRESS_TABLE_PREFIX=wp_ \

  # The following are optional

  WORDPRESS_AUTH_KEY=REPLACE_ME \
  WORDPRESS_SECURE_AUTH_KEY=REPLACE_ME \
  WORDPRESS_LOGGED_IN_KEY=REPLACE_ME \
  WORDPRESS_NONCE_KEY=REPLACE_ME \
  WORDPRESS_AUTH_SALT=REPLACE_ME \
  WORDPRESS_SECURE_AUTH_SALT=REPLACE_ME \
  WORDPRESS_LOGGED_IN_SALT=REPLACE_ME \
  WORDPRESS_NONCE_SALT=REPLACE_ME \

  WORDPRESS_DEBUG=false \
  WORDPRESS_CONFIG_EXTRA= \
```

### Port mapping

```bash
# dokku proxy:set my-wordpress nginx

dokku proxy:ports-add my-wordpress http:80:80 https:443:80

dokku proxy:build-config my-wordpress
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
memory_limit = 256M
max_execution_time = 300
upload_max_filesize = 128M
post_max_size = 128M
```

Mount the file to your app

```bash
dokku storage:mount my-wordpress /var/lib/dokku/data/storage/my-wordpress/config/php-config.ini:/usr/local/etc/php/conf.d/99-php-config.ini
```

### Deploy your app

```bash
dokku git:from-image my-wordpress wordpress:6.1.1-php8.1
```
