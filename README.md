Entendido, aquí tienes el script actualizado para instalar PHP 8.2 y configurar WordPress con Nginx, MySQL, SSL automático y WP-CLI en tu instancia de AWS EC2 con Ubuntu 24.04 LTS:

Guarda este script como `setup_wordpress.sh` y ejecútalo con permisos de superusuario (`sudo bash setup_wordpress.sh`).

```bash
#!/bin/bash

# Actualizar el sistema
sudo apt update && sudo apt upgrade -y

# Instalar Nginx
sudo apt install nginx -y

# Instalar MySQL
sudo apt install mysql-server -y

# Configurar MySQL
sudo mysql_secure_installation <<EOF

y
0
y
y
y
y
EOF

# Crear base de datos y usuario para WordPress
sudo mysql -u root <<EOF
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
EOF

# Instalar PHP 8.2
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install php8.2-fpm php8.2-mysql -y

# Descargar y configurar WordPress
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
sudo cp -a /tmp/wordpress/. /var/www/html

# Configurar archivo wp-config.php
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
sudo sed -i "s/database_name_here/wordpress/" wp-config.php
sudo sed -i "s/username_here/wordpressuser/" wp-config.php
sudo sed -i "s/password_here/password/" wp-config.php

# Ajustar permisos
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 750 {} \;
sudo find /var/www/html -type f -exec chmod 640 {} \;

# Configurar Nginx
cat <<EOF | sudo tee /etc/nginx/sites-available/wordpress
server {
    listen 80;
    server_name domine.com;

    root /var/www/html;
    index index.php index.html index.htm;

    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }

    location ~ \.php\$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
EOF

# Habilitar configuración de Nginx y reiniciar
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Instalar Certbot para SSL
sudo apt install certbot python3-certbot-nginx -y

# Obtener y configurar el certificado SSL
sudo certbot --nginx -d petloversapp.com --non-interactive --agree-tos -m tu_email@example.com

# Verificar auto-renovación de SSL
sudo systemctl status certbot.timer
sudo certbot renew --dry-run

# Instalar WP-CLI
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp

# Completar la instalación de WordPress
sudo -u www-data -- wp core install --url="https://petloversapp.com/" --title="Pet Lovers App" --admin_user="admin" --admin_password="password" --admin_email="petloversapp2024@gmail.com"

echo "Instalación completada. Por favor, visita http://domine.com para verificar."
```

Para ejecutar el script, sigue estos pasos:

1. Guarda el script en un archivo llamado `setup_wordpress.sh`.
2. Otorga permisos de ejecución al archivo:

    ```bash
    chmod +x setup_wordpress.sh
    ```

3. Ejecuta el script como superusuario:

    ```bash
    sudo bash setup_wordpress.sh
    ```

Este script instalará y configurará WordPress con Nginx, MySQL, PHP 8.2, un certificado SSL automático y WP-CLI en tu instancia de AWS EC2. Asegúrate de reemplazar `tu_email@example.com` y las credenciales de administrador de WordPress con tus propios valores.

Instalar addicionalmente los modulos de PHP necesarios
