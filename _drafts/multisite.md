https://github.com/pie-inc/docker-wordpress-multisite
https://github.com/docker-library/wordpress/blob/940a0d35951a0917622c35acc92b38b1db3c730f/latest/php8.0/apache/Dockerfile

# enable multisite


# fix wp-content permissions
chown -R 33:33 *
find . -type d -exec chmod 755 {} \;  # Change directory permissions rwxr-xr-x
find . -type f -exec chmod 644 {} \;  # Change file permissions rw-r--r--