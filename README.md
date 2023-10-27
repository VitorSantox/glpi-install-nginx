
# Instalação do GLPI - Passo a Passo

## 1.0 Pré-Requisitos

Para instalar o GLPI, você precisará atender a alguns pré-requisitos:

- 1.1 Servidor Ubuntu 22.04 LTS Instalado
- 1.2 Acesso à internet para download e instalação de pacotes
- 1.3 Acesso de root ao servidor

## 2.0 Instalação

Vamos prosseguir com a instalação do GLPI:

- 2.1 Atualizar o sistema:
  ```bash
  sudo apt update && sudo apt dist-upgrade -y
  ```

- 2.2 Reconfigurar o timezone:
  ```bash
  sudo dpkg-reconfigure tzdata
  ```

- 2.3 Reiniciar o servidor:
  ```bash
  sudo reboot
  ```

- 2.4 Instalar Apache, PHP e MySQL:
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

- 2.5 Criar o banco de dados do GLPI:
  ```bash
  sudo mysql -e "CREATE DATABASE glpi"
  sudo mysql -e "GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost' IDENTIFIED BY 'P4ssw0rd'"
  sudo mysql -e "GRANT SELECT ON mysql.time_zone_name TO 'glpi'@'localhost'"
  sudo mysql -e "FLUSH PRIVILEGES"
  ```

- 2.6 Carregar timezones no MySQL:
  ```bash
  mysql_tzinfo_to_sql /usr/share/zoneinfo | sudo mysql -u root mysql
  ```

- 2.7 Desabilitar o site padrão do Apache:
  ```bash
  sudo a2dissite 000-default.conf
  ```

## 3.0 Configuração do Nginx (Alterações para usar o Nginx)

Aqui estão as etapas específicas para configurar o Nginx como servidor web:

- 3.1 Instalar o Nginx:
  ```bash
  sudo apt install nginx
  ```

- 3.2 Configurar o Virtual Host do Nginx:
  ```bash
  sudo nano /etc/nginx/sites-available/glpi
  ```

  Dentro do arquivo, insira a configuração do Virtual Host do Nginx, como exemplificado anteriormente.

- 3.3 Habilitar o Virtual Host do Nginx:
  ```bash
  sudo ln -s /etc/nginx/sites-available/glpi /etc/nginx/sites-enabled/
  ```

  Reinicie o Nginx para que as alterações tenham efeito:
  ```bash
  sudo systemctl restart nginx
  ```

## 4.0 Ajustes de Segurança

Essas etapas permanecem as mesmas, independentemente do servidor web que você esteja usando:

- 4.1 Remover o arquivo de instalação:
  ```bash
  sudo rm /var/www/glpi/install/install.php
  ```

- 4.2 Mover pastas do GLPI de forma segura:
  ```bash
  sudo mv /var/www/glpi/files /var/lib/glpi
  sudo mv /var/www/glpi/config /etc/glpi
  sudo mkdir /var/log/glpi && sudo chown -R www-data:www-data /var/log/glpi
  ```

- 4.3 Mover pastas do GLPI de forma segura | conf-dir:
  ```bash
  sudo nano /var/www/glpi/inc/downstream.php
  ```

  Dentro do arquivo, insira o código conforme mostrado na etapa 3.3 no passo a passo original.

- 4.4 Mover pastas do GLPI de forma segura | data dir:
  ```bash
  sudo nano /etc/glpi/local_define.php
  ```

  Insira o código conforme mostrado na etapa 3.4 no passo a passo original.

Agora, você deve ter o GLPI configurado e seguro, com o Nginx como servidor web.
```

Espero que esta formatação em markdown torne o passo a passo mais claro e fácil de seguir. Certifique-se de adaptar as configurações de acordo com o seu ambiente específico. Se tiver mais alguma pergunta ou precisar de esclarecimentos adicionais, sinta-se à vontade para perguntar.
