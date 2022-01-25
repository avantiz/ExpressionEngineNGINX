# COMO INSTALAR O EXPRESSIONENGINE EM UBUNTU 18 LTS E NGINX

O ExpressionEngine precisa de um webserver executando PHP e MySQL. No momento em que escrevo este tutorial, ele precisa, no mínimo:

- PHP 7.0 ou mais recente, com as seguintes extensões:
	 - gd
	 - fileinfo
	 - intl
	 - mbstring

- MySQL 5.6 ou mais recente ou Percona 5.6 ou mais recente. Neste guia vamos utilizar o MySQL.
- Um servidor web usando Apache ou NGINX. Neste guia vamos utilizar o NGINX.

# Antes de começar:

Verifique a sua versão do Ubuntu
```
lsb_release -ds
# Ubuntu 18.04.2 LTS
```
Crie uma conta non-root com sudo e troque para ela:
```
adduser fulano --gecos "Fulano"
usermod -aG sudo fulano  
su - fulano
```
NOTA: troque fulano pelo seu nome de usuário

Defina o fuso horário:
```
sudo dpkg-reconfigure tzdata
```
Tenha certeza que seu sistema está atualizado:
```
sudo apt update && sudo apt upgrade -y
```
Instale os pacotes que forem necessários:
```
sudo apt install -y zip unzip curl wget git
```
# Instale o PHP

Instale o PHP, assim como as extensões que são necessárias:
```
sudo apt install -y php7.4 php7.4-cli php7.4-fpm php7.4-common php7.4-mbstring php7.4-gd php7.4-intl php7.4-mysql
```
Confira a versão:
```
php --version
```
# Instale o MYSQL

Instale o MySQL, a seguir:
```
sudo apt install -y mysql-server
```
Confira a versão:
```
mysql --version
```
Execute o script mysql_secure_installation para incrementar sua instalação MySQL:
```
sudo mysql_secure_installation
```
Efetue o login no MySQL como usuário root:
```
sudo mysql -u root -p
# Enter password:
```
Crie um novo banco de dados e um novo usuário no banco de dados e anote as credenciais:
```
mysql> CREATE DATABASE seubancodedados;
mysql> GRANT ALL ON seubancodedados.* TO 'usuáriobancodedados' IDENTIFIED BY 'suasenhadobancodedados';
mysql> FLUSH PRIVILEGES;
mysql> quit
```
NOTA: Substitua "seubancodados", "usuáriobancodedados" e "suasenhadobancodedados" pelos seus dados. Use uma senha segura.

# Instale o NGINX:

Instale o servidor web Nginx com o seguinte comando:
```
sudo apt install -y nginx
```
Verifique a versão:
```
sudo nginx -v
```
Agora vamos configurar o Nginx para o ExpressionEngine. Execute 
``` sudo vim /etc/nginx/sites-available/expressionengine.conf ``` e preencha o arquivo com a seguinte configuração:

```
server {

  listen [::]:80;
  listen 80;

  server_name example.com;
  root /var/www/expressionengine;

  index index.php;

  location / {
    index index.php;
    try_files $uri $uri/ @ee;
  }

  location @ee {
    rewrite ^(.*) /index.php?$1 last;
  }

  location ~* \.php$ {
    fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    include fastcgi_params;
    fastcgi_index index.php5;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  }

}

```

Salve o arquivo e saia com ```:``` + ```W``` + ```Q```

Ative a nova configuração ```expressionengine.conf``` criada, linkando o arquivo à pasta ```sites-enabled```.

Teste a configuração:

``` sudo nginx -t ```

Recarregue o Nginx:
```
sudo systemctl reload nginx.service
```

# Instalando o ExpressionEngine

Crie uma pasta na raiz:
```
sudo mkdir -p /var/www/expressionengine
```
Altere o proprietário da pasta /var/www/expressionengine  para fulano (o nome de usuário que você escolheu lá em cima):
```
sudo chown -R fulano:fulano /var/www/expressionengine
```
Navegue para a pasta root:
```
cd /var/www/expressionengine
```
Baixe a última versão do ExpressionEngine e descompacte os arquivos para uma pasta no seu servidor:
```
wget -O ee.zip --referer https://expressionengine.com/ 'https://expressionengine.com/?ACT=243'
unzip ee.zip
rm ee.zip
```

Altere o proprietário da pasta ``` /var/www/expressionengine ``` para ```www-data```:
```
sudo chown -R www-data:www-data /var/www/expressionengine
```

Aponte seu navegador para a URL do arquivo ```admin.php``` do ExpressionEngine. Por exemplo: ```https://seusite.com/admin.php``` e siga as instruções para instalar o ExpressionEngine. Assim que você finalizar a instalação, remova a pasta ```system/ee/installer/``` do seu servidor.
