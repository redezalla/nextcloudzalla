# nextcloudzalla
nextcloud up
NextCloud
#Sobre
Olá! Visitante, reunimos aqui, informações sobre a ferramenta NextCloud, esta ferramenta possibilita a criação de um sistema de compartilhamento de arquivos baseado nos modelos de Nuvem, ou seja, você pode compartilhar arquivos como os serviços oferecidos pelo DropBox, Google Drive e diversos outros, além de ter acesso a muitos recursos e AddOns que deixam a ferramenta com suporte a diversas funções.



#Site Oficial: https://nextcloud.com/

#Doc: https://docs.nextcloud.com/server/16/admin_manual/contents.html


Instalando dependências
 apt-get install apache2 mariadb-server php7.3-xml php7.3 libapache2-mod-php7.3 php7.3-cgi php7.3-cli php7.3-gd php7.3-curl php7.3-apcu php7.3-zip php-ldap php7.3-mysql php-intl php7.3-intl php7.3-mbstring memcached wget unzip
Configurando BD (MariaDB)

 mysql_secure_installation

 mysql -u root -p

  CREATE DATABASE nextclouddb;
  CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'senhasegura';
  GRANT ALL PRIVILEGES ON nextclouddb.* TO 'nextcloud'@'localhost';
  FLUSH PRIVILEGES;
  QUIT

Baixando NextCloud
 wget https://download.nextcloud.com/server/releases/latest.zip
Extração e Ajustes de pastas
 rm -rf /var/www/html/index.html
 unzip latest.zip
 mv nextcloud /var/www/html/
 chown -R www-data:www-data /var/www/html/nextcloud

Criar arquivo de Conf
 nano /etc/apache2/sites-available/nextcloud.conf
 <VirtualHost *:80>
 ServerAdmin nextcloud@projetoroot.com.br
 DocumentRoot "/var/www/html/nextcloud"
 ServerName nextcloud
 <Directory "/var/www/html/nextcloud/">
 Options MultiViews FollowSymlinks
 
 AllowOverride All
 Order allow,deny
 Allow from all
 </Directory>
 TransferLog /var/log/apache2/nextcloud_access.log
 ErrorLog /var/log/apache2/nextcloud_error.log
 </VirtualHost>
Ajustando apache
 a2dissite 000-default
 a2ensite nextcloud
 systemctl restart apache2
Se tiver problemas ao utilizar os comandos acima, faça:
 nano /etc/profile
Edite a primeira linha do PATH deixando assim


PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/sbin
Após execute

 . /etc/profile
Utilizando o OPCache
 nano /etc/php/7.0/apache2/php.ini
 opcache.enable=1
 opcache.enable_cli=1
 opcache.interned_strings_buffer=8
 opcache.max_accelerated_files=10000
 opcache.memory_consumption=128
 opcache.save_comments=1
 opcache.revalidate_freq=1
 nano /var/www/html/nextcloud/config/config.php
 'memcache.local' => '\OC\Memcache\APCu',

Instalando Cliente (outro computador/vm/mobile)
PC
Linux(Debian/Ubuntu)

 add-apt-repository ppa:nextcloud-devs/client
 apt-get update -y
 apt-get install nextcloud-client -y
 Windows: https://download.nextcloud.com/desktop/releases/Windows/Nextcloud-2.5.3-setup.exe
 MacOS: https://download.nextcloud.com/desktop/releases/Mac/Installer/Nextcloud-2.5.3.20190724.pkg
Mobile
 Android: https://play.google.com/store/apps/details?id=com.nextcloud.client
 Iphone: https://itunes.apple.com/us/app/nextcloud/id1125420102?mt=8
 Windows: https://www.microsoft.com/store/apps/9nblggh532xq
Aumentando a Segurança no NextCloud



Configurar NextCloud para SSL
1 - Editando config e adicionando parâmetros para forçar o ssl

 nano /var/www/html/nextcloud/config/config.php
 'forcessl' => true,
 'forceSSLforSubdomains' => true,

Redirecionar HTTP para HTTPS
1 - Editar nexcloud.conf e adicionar o redirect

  nano /etc/apache2/sites-available/nextcloud.conf
  
  Redirect permanent / https://192.168.201.100/
IP 192.168.201.100 é o IP do Nextcloud mostrado no vídeo, configure o endereço do seu NextCloud
Gerar Certificado Auto Assinado
1 - Execute o comando abaixo para gerar um repositório de chaves chamado “nextcloudauto.key”

  openssl genrsa -out nextcloudauto.key 2048
2 - Execute o comando abaixo para alterar a permissão do repositório (600 apenas o dono pode ler e escrever)

  chmod 600 nextcloudauto.key
3 - Execute os comandos abaixo para requisitar um novo certificado SSL

  openssl req -new -key nextcloudauto.key -out nextcloudauto.csr
  openssl x509 -req -days 365 -in nextcloudauto.csr -signkey nextcloudauto.key -out nextcloudauto.crt
4 - Execute o comando abaixo para copiar as chaves e certificados para os seus respectivos caminhos

  cp nextcloudauto.key /etc/ssl/private/ && cp nextcloudauto.crt /etc/ssl/certs/
Habilitar módulos no apache
1 - Execute o comando abaixo para habilitar os modulos ssl e headers

  a2enmod ssl && a2enmod headers
2 - Edite o arquivo .conf e deixe conforme exemplo

  nano /etc/apache2/sites-available/nextcloud-ssl.conf

  <IfModule mod_ssl.c>
          <VirtualHost _default_:443>
                  ServerAdmin nextcloud@projetoroot.com.br
                  ServerName nextcloud
                  DocumentRoot "/var/www/html/nextcloud"
                  Options MultiViews FollowSymlinks
  
                  SSLEngine on
    
                  # Para Utilizar o LetsEncrypt apenas alterar o caminho conforme exemplo:
  
                  ## SSLCertificateFile /etc/letsencrypt/live/dominio.com.br/fullchain.pem
                  ## SSLCertificateKeyFile /etc/letsencrypt/live/dominio.com.br/privkey.pem
                  
                  SSLCertificateFile      /etc/ssl/certs/nextcloudauto.crt
                  SSLCertificateKeyFile /etc/ssl/private/nextcloudauto.key
  
                  <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                  SSLOptions +StdEnvVars
                  </FilesMatch>
                  <Directory /usr/lib/cgi-bin>
                                  SSLOptions +StdEnvVars
                   AllowOverride All
                   Order allow,deny
                   Allow from all
                   </Directory>
   TransferLog /var/log/apache2/nextcloud_access.log
   ErrorLog /var/log/apache2/nextcloud_error.log
  
  <IfModule mod_headers.c>
        Header always set Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"
        Header Set X-Robots-Tag "none"
        Header Set X-Content-Type-Options nosniff
        Header Set X-Frame-Options "SAMEORIGIN"
        Header set X-XSS-Protection "1; mode=block"
  
      <FilesMatch "\.(%FilesMatchPattern%)$">
          Header unset X-XSS-Protection
      </FilesMatch>
  
   
      </IfModule>
  
  
          </VirtualHost>
  
  
  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet

Habilitar nova conf
3 - Execute o comando abaixo para habilitar a nova conf

  a2ensite nextcloud-ssl.conf
4 - Execute o comando abaixo para reiniciar o serviço de HTTP

  /etc/init.d/apache2 restart
Arquitetura ARM - Raspberry PI 3
  *Em Breve
