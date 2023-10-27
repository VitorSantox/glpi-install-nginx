
# GLPI-Install
## Script de Instalação do GLPI

### 1. Pré-Requisitos

- Servidor Ubuntu 22.04 LTS Instalado
- Acesso à internet para download e instalação de pacotes
- Acesso de root ao servidor

### 2. Instalação

2.1. Atualizar
```bash
sudo apt update && sudo apt dist-upgrade -y
```

2.2. Reconfigurar timezone
```bash
sudo dpkg-reconfigure tzdata
```

2.3. Reiniciar
```bash
sudo reboot
```

2.4. Instalar Apache, PHP e MySQL
```bash
sudo apt install -y \
	apache2 \
	mariadb-server \
	mariadb-client \
	libapache2-mod-php \
	php-dom \
	php-fileinfo   \
	php-json \
	php-simplexml \
	php-xmlreader \
	php-xmlwriter \
	php-curl \
	php-gd \
	php-intl \
	php-mysqli   \
	php-bz2  \
	php-zip \
	php-exif \
	php-ldap  \
	php-opcache \
	php-mbstring
```

2.5. Criar banco de dados do GLPI
```bash
sudo mysql -e "CREATE DATABASE glpi"
sudo mysql -e "GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost' IDENTIFIED BY 'P4ssw0rd'"
sudo mysql -e "GRANT SELECT ON mysql.time_zone_name TO 'glpi'@'localhost'"
sudo mysql -e "FLUSH PRIVILEGES"
```

2.6. Carregar timezones no MySQL
```bash
mysql_tzinfo_to_sql /usr/share/zoneinfo | sudo mysql -u root mysql
```

2.7. Desabilitar o site padrão do apache2
```bash
sudo a2dissite 000-default.conf
```

2.8. Habilitar `session.cookie_httponly`
```bash
sudo sed -i 's/^session.cookie_httponly =/session.cookie_httponly = on/' /etc/php/8.1/apache2/php.ini && \
	sudo sed -i 's/^;date.timezone =/date.timezone = America/Sao_Paulo/' /etc/php/8.1/apache2/php.ini
```

2.9. Criar o virtualhost do GLPI
```bash
cat << EOF | sudo tee /etc/apache2/sites-available/glpi.conf
<VirtualHost *:80>
	ServerName glpi.ninjapfsense.com.br
	DocumentRoot /var/www/glpi/public
	<Directory /var/www/glpi/public>
		Require all granted
		RewriteEngine On
		# Redirect all requests to GLPI router, unless file exists.
		RewriteCond %{REQUEST_FILENAME} !-f
		RewriteRule ^(.*)$ index.php [QSA,L]
	</Directory>
</VirtualHost>
EOF
```

2.10. Habilitar o virtualhost
```bash
sudo a2ensite glpi.conf
```

2.11. Habilitar módulos do Apache necessários
```bash
sudo a2enmod rewrite
```

2.12. Reiniciar o Apache para que as alterações entrem em vigor
```bash
sudo systemctl restart apache2
```

2.13. Download do GLPI
```bash
wget -q https://github.com/glpi-project/glpi/releases/download/10.0.7/glpi-10.0.7.tgz
```

2.14. Descompactar a pasta do GLPI
```bash
tar -zxf glpi-*
```

2.15. Mover a pasta do GLPI para a pasta htdocs
```bash
sudo mv glpi /var/www/glpi
```

2.16. Configurar as permissões na pasta www/glpi
```bash
sudo chown -R www-data:www-data /var/www/glpi/
```

2.17. Finalizar a configuração do GLPI pela linha de comando
```bash
sudo php /var/www/glpi/bin/console db:install \
	--default-language=pt_BR \
	--db-host=localhost \
	--db-port=3306 \
	--db-name=glpi \
	--db-user=glpi \
	--db-password=P4ssw0rd
```

### 3. Ajustes de Segurança

3.1. Remover o arquivo de instalação
```bash
sudo rm /var/www/glpi/install/install.php
```

3.2. Mover pastas do GLPI de forma segura
```bash
sudo mv /var/www/glpi/files /var/lib/glpi
sudo mv /var/www/glpi/config /etc/glpi
sudo mkdir /var/log/glpi && sudo chown -R www-data:www-data /var/log/glpi
```

3.3. Mover pastas do GLPI de forma segura (conf-dir)
```bash
cat << EOF | sudo tee /var/www/glpi/inc/downstream.php
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
   require_once GLPI_CONFIG_DIR . '/local_define.php';
}
EOF
```

3.4. Mover pastas do GLPI de forma segura (data dir)
```bash
cat << EOF | sudo tee /etc/glpi/local_define.php
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_LOG_DIR', '/var/log/glpi');
EOF
```

### 4. Primeiros Passos

4.1. Acessar o GLPI via web browser
4.2. Criar um novo usuário com perfil super-admin
4.3. Remover os usuários glpi, normal, post-only, tech.
4.3.1. Enviar os usuários para a lixeira
4.3.2. Remover permanentemente
4.3.4. Configurar a URL de acesso ao sistema em: Configurar -> Geral -> Configuração Geral -> URL da aplicação.

### Informações Adicionais

https://github.com/MasterMindTI/glpi-install
https://glpi-install.readthedocs.io/pt/latest/install/index.html

