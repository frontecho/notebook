# **Deploy WordPress Blog With Docker**

```bash
docker run -d \
  --privileged=true \
  --name wordpressdb \
  --env MYSQL_ROOT_PASSWORD=YOUR_PASSWORD \
  --env MYSQL_DATABASE=wordpress \
  -v "$PWD/wordpressdb":/var/lib/mysql \
  --restart unless-stopped \
   mysql:5.7
```

```bash
docker run -d \
  --name wordpress \
  --link wordpressdb:mysql \
  -e WORDPRESS_DB_PASSWORD=YOUR_PASSWORD \
  -p 127.0.0.1:8080:80 \
  -v "$PWD/wordpress":/var/www/html \
  --restart unless-stopped \
  wordpress:latest
```

```bash
vim wordpress/wp-config.php
```

```php
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', getenv_docker('WORDPRESS_DB_NAME', 'wordpress') );
/** MySQL database username */
define( 'DB_USER', getenv_docker('WORDPRESS_DB_USER', 'root') );
/** MySQL database password */
define( 'DB_PASSWORD', getenv_docker('WORDPRESS_DB_PASSWORD', 'YOUR_PASSWORD') );
```
