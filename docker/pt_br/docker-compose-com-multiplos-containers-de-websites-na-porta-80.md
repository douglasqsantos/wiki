# Docker Compose com multiplos containers de websites na porta 80

Galera, aqui vamos abordar a configuração de containers trabalhando como servidor de multiplos websites na porta 80, como todos sabemos podemos utilizar somente uma **porta 80** por Docker Host, para conseguir contornar esta limitação vamos configurar o **Nginx** como **proxy reverso** dando suporte a duas aplicações: **WordPress** e **Drupal**. Eu escolhi estas duas aplicações pois são bem comuns e em alguns casos vamos precisar ter rodando ambas em um mesmo Docker Host.

**O que eu vou utilizar:**

* Ip do Docker Host: **10.3.0.91**
* Endereço para o Wordpress: **wordpress.dqs.local**
* Endereço para o Drupal: **drupal.dqs.local**
* Docker Compose: **version 1.24.1, build 4667896b**

## Configurando o Docker Compose para o Systemd

Vamos criar o arquivo que vai manipular os serviços do docker-compose

```bash
vim /etc/systemd/system/docker-compose@.service
[Unit]
Description=%i service with docker compose
Requires=docker.service
After=docker.service

[Service]
Restart=always

#Diretório que vai armazenar os arquivos de compose
WorkingDirectory=/srv/docker/compose/%i

# Remove containers antigos, images e volumes
# Sempre tente manter a sua estrutura de docker organizada e sem danglings
#ExecStartPre=/usr/local/bin/docker-compose down -v
#ExecStartPre=/usr/local/bin/docker-compose rm -fv

## Habilitar os comandos abaixo caso queira sempre garantir que não temos dangling volumes/netwoks/containers
#ExecStartPre=-/bin/bash -c 'docker volume ls -qf "name=%i_" | xargs docker volume rm'
#ExecStartPre=-/bin/bash -c 'docker network ls -qf "name=%i_" | xargs docker network rm'
#ExecStartPre=-/bin/bash -c 'docker ps -aqf "name=%i_*" | xargs docker rm'

# Subindo o Compose
ExecStart=/usr/bin/docker-compose up

# Compose down, remove containers e volumes
#ExecStop=/usr/bin/docker-compose down -v

# Parando o Compose
ExecStop=/usr/bin/docker-compose down

[Install]
WantedBy=multi-user.target
```

Agora vamos criar o diretório para armazenar os arquivos de compose

```bash
mkdir -p /srv/docker/compose/
```

Agora vamos criar o diretório para armazenar os dados dos containers

```bash
mkdir -p /srv/docker/data
```

Agora vamos criar um serviço para testar, vamos começar criando o diretório para armazenar o nosso novo serviço.

```bash
mkdir -p /srv/docker/compose/proxy_reverso
```

## Configurando o docker-compose.yml

Agora vamos criar o nosso docker-compose file que vai controlar os serviços: Nginx, Wordpress, MySQL, Drupal e PostgreSQL.

```yaml
vim /srv/docker/compose/proxy_reverso/docker-compose.yml
version: '3'
services:
  reverseproxy:
    image: reverseproxy:v1
    build:
      context: '.'
      dockerfile: Dockerfile
    ports:
      - 80:80
    networks:
      - wp_net
      - drupal_net
    restart: always
  db-wp:
    container_name: db-wp
    image: mysql:5.7
    volumes:
      - ${DATA_DIR_MYSQL-/srv/docker/data/wordpress/data}:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    networks:
      - wp_net
    restart: always
  wp-web:
    container_name: wp-web
    volumes:
      - ${DATA_DIR_WP-/srv/docker/data/wordpress/html}:/var/www/html
    depends_on:
      - reverseproxy
      - db-wp
    image: wordpress
    environment:
      WORDPRESS_DB_HOST: db-wp:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    networks:
      - wp_net
    restart: always
  drupal:
    container_name: drupal-web
    image: drupal:8-apache
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-themes:/var/www/html/themes
      - drupal-sites:/var/www/html/sites
    depends_on:
      - reverseproxy
      - db-drupal
    networks:
      - drupal_net
    restart: always
  db-drupal:
    container_name: db-drupal
    image: postgres:10
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - drupal_net
    restart: always
volumes:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:
  db_data:
networks:
  wp_net:
  drupal_net:
```

Aqui temos 5 serviços:

* **reverseproxy**:
  * Vai ser gerado do Dockerfile
  * Expoe a porta 80 para que possamos utilizar o serviço do Nginx como proxy
  * Utiliza a network do wordpress e do drupal
* **db-wp**:
  * Utiliza a versão 5.7 do MySQL
  * Armazena os dados em um volume pré definido
  * Temos as configurações básicas do MySQL
  * Utiliza a network wp_net
* **wp-web**:
  * Utiliza a ultima versão do Wordpress
  * Armazena os dados em um volume pré definido
  * Depende do reverseproxy e do db-wp
  * Temos as configurações básicas do BD para o Wordpress.
  * Utiliza a network wp_net
* **drupal**:
  * Utiliza a ultima versão do 8 do Drupal
  * Armazena os dados em volumes definidos pelo Docker
  * Depende do reverseproxy e do db-drupal
  * Utiliza a network drupal_net
* **db-drupal**:
  * Utiliza a ultima versão do 10 do postgresql
  * Armazena os dados em volumes definidos pelo Docker
  * Utiliza a network drupal_net

Agora vamos criar o Dockerfile para o Nginx

```dockerfile
vim /srv/docker/compose/proxy_reverso/Dockerfile
# Vamos utilizar o Nginx do Alpine
FROM nginx:alpine
# Arquivo de configuração para o Nginx
COPY nginx.conf /etc/nginx/nginx.conf
```

Agora vamos criar o arquivo de controle do Nginx

```bash
vim /srv/docker/compose/proxy_reverso/nginx.conf
worker_processes 1;

events { worker_connections 1024; }

http {

    sendfile on;

    upstream docker-wordpress {
        server wp-web:80;
    }

    upstream docker-drupal {
        server drupal-web:80;
    }

    server {
        listen 80;
        server_name wordpress.dqs.local;

        location / {
            proxy_pass         http://docker-wordpress;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
        }
    }

    server {
        listen 80;
        server_name drupal.dqs.local;

        location / {
            proxy_pass         http://docker-drupal;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
        }
    }
}
```

Aqui temos 2 servidores web que vamos mandar as requisições:

* **docker-wordpress**
  * Container: wp-web
  * Porta: 80
* **docker-drupal**
  * Container: drupal-web
  * Porta: 80

Quando vamos configurar o proxy reverso para websites notem que eu não configurei portas para os servidores web somente para o proxy, desta forma o proxy vai encaminhar as requisições pelas redes que demos acesso a ele sem precisarmos expor as portas para o Docker Host.

Agora vamos testar o nosso docker-compose.yml

```bash
cd /srv/docker/compose/proxy_reverso && docker-compose config
networks:
  drupal_net: {}
  wp_net: {}
services:
  db-drupal:
    container_name: db-drupal
    environment:
      POSTGRES_PASSWORD: example
    image: postgres:10
    networks:
      drupal_net: null
    restart: always
    volumes:
    - db_data:/var/lib/postgresql/data:rw
  db-wp:
    container_name: db-wp
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_USER: wordpress
    image: mysql:5.7
    networks:
      wp_net: null
    restart: always
    volumes:
    - /srv/docker/data/wordpress2/data:/var/lib/mysql:rw
  drupal:
    container_name: drupal-web
    depends_on:
    - db-drupal
    - reverseproxy
    image: drupal:8-apache
    networks:
      drupal_net: null
    restart: always
    volumes:
    - drupal-modules:/var/www/html/modules:rw
    - drupal-profiles:/var/www/html/profiles:rw
    - drupal-themes:/var/www/html/themes:rw
    - drupal-sites:/var/www/html/sites:rw
  reverseproxy:
    build:
      context: /srv/docker/compose/nginx-reverse/reverse_proxy
      dockerfile: Dockerfile
    image: reverseproxy:v1
    networks:
      drupal_net: null
      wp_net: null
    ports:
    - 80:80/tcp
    restart: always
  wp-web:
    container_name: wp-web
    depends_on:
    - db-wp
    - reverseproxy
    environment:
      WORDPRESS_DB_HOST: db-wp:3306
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_USER: wordpress
    image: wordpress
    networks:
      wp_net: null
    restart: always
    volumes:
    - /srv/docker/data/wordpress2/html:/var/www/html:rw
version: '3.0'
volumes:
  db_data: {}
  drupal-modules: {}
  drupal-profiles: {}
  drupal-sites: {}
  drupal-themes: {}
```

Agora vamos execuar o build da imagem do reverseproxy

```bash
cd /srv/docker/compose/proxy_reverso && docker-compose build
db-wp uses an image, skipping
wp-web uses an image, skipping
db-drupal uses an image, skipping
drupal uses an image, skipping
Building reverseproxy
Step 1/2 : FROM nginx:alpine
 ---> 4d3c246dfef2
Step 2/2 : COPY nginx.conf /etc/nginx/nginx.conf
 ---> 249070beb1bc
Successfully built 249070beb1bc
Successfully tagged reverseproxy:v1
```

Agora vamos subir os serviços

```bash
docker-compose up -d
Creating network "reverse_proxy_wp_net" with the default driver
Creating network "reverse_proxy_drupal_net" with the default driver
Creating volume "reverse_proxy_drupal-modules" with default driver
Creating volume "reverse_proxy_drupal-profiles" with default driver
Creating volume "reverse_proxy_drupal-sites" with default driver
Creating volume "reverse_proxy_drupal-themes" with default driver
Creating volume "reverse_proxy_db_data" with default driver
Creating db-wp                        ... done
Creating db-drupal                    ... done
Creating reverse_proxy_reverseproxy_1 ... done
Creating drupal-web                   ... done
Creating wp-web                       ... do
```

Agora vamos listar os serviços, o comando abaixo funciona somente se estiver no mesmo diretório do docker-compose.yml

```bash
docker-compose ps
            Name                          Command               State          Ports       
-------------------------------------------------------------------------------------------
db-drupal                      docker-entrypoint.sh postgres    Up      5432/tcp           
db-wp                          docker-entrypoint.sh mysqld      Up      3306/tcp, 33060/tcp
drupal-web                     docker-php-entrypoint apac ...   Up      80/tcp             
reverse_proxy_reverseproxy_1   nginx -g daemon off;             Up      0.0.0.0:80->80/tcp 
wp-web                         docker-entrypoint.sh apach ...   Up      80/tcp  
```

Agora na máquina cliente (que vai acessar pelo navegador) vamos criar duas entradas no /etc/hosts para podermos testar o proxy reverso. Caso tenha um DNS na rede insira as entradas no seu arquivo de dns.

```bash
sudo vim /etc/hosts
[...]
10.3.0.91 wordpress.dqs.local
10.3.0.91 drupal.dqs.local
```

## Configurando o WordPress

Precisamos acessar: http://wordpress.dqs.local

Configurando:

* Na primeira página
  * Selecione o idioma: English (United States) e selecione Continue
* Na segunda página
  * Site Title: Nome do Site
  * Username: admin
  * Password: 21smvOgCUdRyGzKO8O
  * Your Email: douglas.q.santos@gmail.com
  * Selecione Install Wordpress
* Na terceira página
  * Selecione Log In
* Na quarta página
  * informe o usuário e senha para efetuar o login

## Configurando o Drupal

Agora vamos acessar: http://drupal.dqs.local para configurar o Drupal

* Na primeira página
  * Choose language: Selecione o Idioma (English) e click em Save and Continue
* Na segunda página
  * Select an installation profile: Standard e click em Save and Continue
* Na terceira página
  * Database type: PostgreSQL
  * Database name: postgres
  * Database username: postgres
  * Database password: example
  * Host (under Advanced Options): db-drupal
  * Click em Save and continue
* Na quarta página
  * Site name: Nome do Site
  * Site email address: douglas.q.santos@gmail.com
  * Username: admin
  * Password: 4bwbBaF4Pt62mz4w
  * Confirm password: 4bwbBaF4Pt62mz4w
  * Email address: douglas.q.santos@gmail.com
  * Default country: Brazil
  * Default time zone: Sao Paulo
  * Agora click em Save and continue.

## Habilitando o serviço no Systemd

Agora vamos parar os serviços do Docker

```bash
cd /srv/docker/compose/proxy_reverso && docker-compose down
```

Agora vamos habilitar o serviço do Nginx para o serviço do Docker Compose

```bash
systemctl enable docker-compose@reverse_proxy
```

Para subir o serviços podemos executar

```bash
systemctl start docker-compose@reverse_proxy
```

Agora podemos consultar o status dos nossos serviços

```bash
systemctl status docker-compose@reverse_proxy  -l
● docker-compose@reverse_proxy.service - reverse_proxy service with docker compose
   Loaded: loaded (/etc/systemd/system/docker-compose@.service; enabled; vendor preset: disabled)
   Active: active (running) since Qui 2019-10-10 15:47:55 -03; 9s ago
  Process: 19728 ExecStop=/usr/bin/docker-compose down (code=exited, status=200/CHDIR)
 Main PID: 19749 (docker-compose)
   CGroup: /system.slice/system-docker\x2dcompose.slice/docker-compose@reverse_proxy.service
           ├─19749 /usr/bin/docker-compose up
           └─19750 /usr/bin/docker-compose up

Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: db-drupal       | 2019-10-10 18:47:57.821 UTC [22] LOG:  database system was shut down at 2019-10-10 18:45:23 UTC
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: db-drupal       | 2019-10-10 18:47:57.834 UTC [1] LOG:  database system is ready to accept connections
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: drupal-web      | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 192.168.48.4. Set the 'ServerName' directive globally to suppress this message
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: drupal-web      | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 192.168.48.4. Set the 'ServerName' directive globally to suppress this message
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: drupal-web      | [Thu Oct 10 18:47:59.018666 2019] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.25 (Debian) PHP/7.3.10 configured -- resuming normal operations
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: drupal-web      | [Thu Oct 10 18:47:59.018759 2019] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: wp-web          | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 192.168.32.4. Set the 'ServerName' directive globally to suppress this message
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: wp-web          | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 192.168.32.4. Set the 'ServerName' directive globally to suppress this message
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: wp-web          | [Thu Oct 10 18:47:59.677423 2019] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.38 (Debian) PHP/7.3.10 configured -- resuming normal operations
Out 10 15:47:59 centos7.dqs.local docker-compose[19749]: wp-web          | [Thu Oct 10 18:47:59.677504 2019] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
```

## Referências

* https://www.bogotobogo.com/DevOps/Docker/Docker-Compose-Nginx-Reverse-Proxy-Multiple-Containers.php
* https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching
* https://www.nginx.com/blog/thread-pools-boost-performance-9x/
* https://www.douglasqsantos.com.br/doku.php/docker_compose_como_um_servico_do_systemd
