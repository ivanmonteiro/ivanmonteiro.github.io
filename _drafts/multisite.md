https://github.com/pie-inc/docker-wordpress-multisite
https://github.com/docker-library/wordpress/blob/940a0d35951a0917622c35acc92b38b1db3c730f/latest/php8.0/apache/Dockerfile

# install wp-cli
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp
wp --info

# enable multisite


# fix wp-content permissions
chown -R 33:33 wp-content/
find wp-content/ -type d -exec chmod 755 {} \;  # Change directory permissions rwxr-xr-x
find wp-content/ -type f -exec chmod 644 {} \;  # Change file permissions rw-r--r--