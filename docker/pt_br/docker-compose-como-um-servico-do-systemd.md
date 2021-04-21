# Docker Compose como um serviço do Systemd

Galera aqui vamos ver a configuração de serviços do Docker Compose com o systemd, em muitos casos precisamos que os serviços subam junto com o SO ou queremos manipular eles como serviços do systemd, desta forma precisamos preparar os arquivos de manipulação do systemd para esta tarefa.

**O que eu vou utilizar:**

* Diretório para armazenar os docker-compose.yml: **/srv/docker/compose**
* Diretório para armazenar os dados dos containers: **/srv/docker/data**
* IP do Docker Host: **10.3.0.91**
* App que vamos utilizar para testar: **Wordpress + MySQL**

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
mkdir -p /srv/docker/compose/wordpress
```

Agora vamos criar o nosso docker-compose file para o wordpress

```yaml
vim /srv/docker/compose/wordpress/docker-compose.yml
version: '3'
services:
  db:
    container_name: wp-mysql
    image: mysql:5.7
    volumes:
      - ${DATA_DIR_MYSQL-/srv/docker/data/wordpress/data}:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    ports:
      - "3306:3306"
    networks:
      - my_net
  wp:
    container_name: wp-web
    volumes:
      - ${DATA_DIR_WP-/srv/docker/data/wordpress/html}:/var/www/html
    depends_on:
      - db
    image: wordpress
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    networks:
      - my_net
networks:
  my_net:
```

Agora vamos criar o arquivo .env

```bash
vim /srv/docker/compose/wordpress/.env
# Local para armazenar os dados do Wordpress
DATA_DIR_WP=/srv/docker/data/wordpress/html
# Local para armazenar os dados do MySQL
DATA_DIR_MYSQL=/srv/docker/data/wordpress/data
```

Agora vamos testar o nosso docker-compose.yml

```bash
cd /srv/docker/compose/wordpress && docker-compose config
networks:
  my_net: {}
services:
  db:
    container_name: wp-mysql
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_USER: wordpress
    image: mysql:5.7
    networks:
      my_net: null
    ports:
    - 3306:3306/tcp
    volumes:
    - /srv/docker/data/wordpress/data:/var/lib/mysql:rw
  wp:
    container_name: wp-web
    depends_on:
    - db
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_USER: wordpress
    image: wordpress
    networks:
      my_net: null
    ports:
    - 80:80/tcp
    volumes:
    - /srv/docker/data/wordpress/html:/var/www/html:rw
version: '3.0'
```

Caso de algum erro acima valide o arquivo, geralmente espaços ou tabs geram problemas.

Caso precise de um arquivo de configuração do vim para trabalhar com yml

```bash
mv ~/.vimrc ~/.vimrc.bkp
```

Agora vamos obter o novo arquivo de configuração

```bash
wget -c https://raw.githubusercontent.com/douglasqsantos/scripts_shell/master/olds/vimrc -O ~/.vimrc
```

Agora vamos criar os diretórios que vão armazenar os dados do wodpress

```bash
mkdir -p /srv/docker/data/wordpress/html
mkdir -p /srv/docker/data/wordpress/data
```

Agora vamos habilitar o serviço do wordpress para o serviço do Docker Compose

```bash
systemctl enable docker-compose@wordpress
```

Agora vamos subir o serviço

```bash
systemctl start docker-compose@wordpress
```

**Nota:** Aqui estamos trabalhando com portas padrões dos serviços: 3306 e 80 então caso tenha algum container trabalhando nestas portas precisa alterar o docker-compose.yml ou parar os containers com as mesmas portas para não gerar conflitos.

Agora vamos vamos verificar o nosso serviço

```bash
systemctl status docker-compose@wordpress -l
● docker-compose@wordpress.service - wordpress service with docker compose
   Loaded: loaded (/etc/systemd/system/docker-compose@.service; enabled; vendor preset: disabled)
   Active: active (running) since Qui 2019-10-10 11:31:18 -03; 29s ago
  Process: 29240 ExecStop=/usr/bin/docker-compose down (code=exited, status=0/SUCCESS)
 Main PID: 29402 (docker-compose)
   CGroup: /system.slice/system-docker\x2dcompose.slice/docker-compose@wordpress.service
           ├─29402 /usr/bin/docker-compose up
           └─29403 /usr/bin/docker-compose up

Out 10 11:31:33 centos7.dqs.local docker-compose[29402]: wp-mysql | 2019-10-10T14:31:33.926861Z 0 [Note]   - '::' resolves to '::';
Out 10 11:31:33 centos7.dqs.local docker-compose[29402]: wp-mysql | 2019-10-10T14:31:33.927006Z 0 [Note] Server socket created on IP: '::'.
Out 10 11:31:33 centos7.dqs.local docker-compose[29402]: wp-mysql | 2019-10-10T14:31:33.943536Z 0 [Warning] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
Out 10 11:31:33 centos7.dqs.local docker-compose[29402]: wp-mysql | 2019-10-10T14:31:33.954892Z 0 [Note] Event Scheduler: Loaded 0 events
Out 10 11:31:33 centos7.dqs.local docker-compose[29402]: wp-mysql | 2019-10-10T14:31:33.955557Z 0 [Note] mysqld: ready for connections.
Out 10 11:31:33 centos7.dqs.local docker-compose[29402]: wp-mysql | Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
Out 10 11:31:36 centos7.dqs.local docker-compose[29402]: wp-web | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 192.168.160.3. Set the 'ServerName' directive globally to suppress this message
Out 10 11:31:36 centos7.dqs.local docker-compose[29402]: wp-web | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 192.168.160.3. Set the 'ServerName' directive globally to suppress this message
Out 10 11:31:36 centos7.dqs.local docker-compose[29402]: wp-web | [Thu Oct 10 14:31:36.891482 2019] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.38 (Debian) PHP/7.3.10 configured -- resuming normal operations
Out 10 11:31:36 centos7.dqs.local docker-compose[29402]: wp-web | [Thu Oct 10 14:31:36.891566 2019] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
```

Agora vamos listar os serviços via compose

```bash
cd /srv/docker/compose/wordpress/ && docker-compose ps
  Name                Command               State                 Ports              
-------------------------------------------------------------------------------------
wp-mysql   docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp, 33060/tcp
wp-web     docker-entrypoint.sh apach ...   Up      0.0.0.0:80->80/tcp
```

Podemos consultar os serviços via docker ps

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
78601c7b90dc        wordpress           "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        0.0.0.0:80->80/tcp                  wp-web
f4cb74224fcd        mysql:5.7           "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp   wp-mysql
```

Agora precisamos acessar: http://10.3.0.91/wp-admin/install.php e configurar o Wordpress.

Após efetuar a configuração vamos parar o serviço do wordpress

```bash
systemctl stop docker-compose@wordpress
```

Agora vamos consultar o status do serviço

```bash
systemctl status docker-compose@wordpress -l
● docker-compose@wordpress.service - wordpress service with docker compose
   Loaded: loaded (/etc/systemd/system/docker-compose@.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Qui 2019-10-10 11:36:59 -03; 22s ago
  Process: 30104 ExecStop=/usr/bin/docker-compose down (code=exited, status=0/SUCCESS)
  Process: 29402 ExecStart=/usr/bin/docker-compose up (code=exited, status=0/SUCCESS)
 Main PID: 29402 (code=exited, status=0/SUCCESS)

Out 10 11:36:59 centos7.dqs.local docker-compose[29402]: wp-mysql | 2019-10-10T14:36:59.034113Z 0 [Note] Shutting down plugin 'mysql_native_password'
Out 10 11:36:59 centos7.dqs.local docker-compose[29402]: wp-mysql | 2019-10-10T14:36:59.034302Z 0 [Note] Shutting down plugin 'binlog'
Out 10 11:36:59 centos7.dqs.local docker-compose[29402]: wp-mysql | 2019-10-10T14:36:59.036576Z 0 [Note] mysqld: Shutdown complete
Out 10 11:36:59 centos7.dqs.local docker-compose[29402]: wp-mysql |
Out 10 11:36:59 centos7.dqs.local docker-compose[29402]: wp-mysql exited with code 0
Out 10 11:36:59 centos7.dqs.local docker-compose[30104]: [101B blob data]
Out 10 11:36:59 centos7.dqs.local docker-compose[30104]: Removing wp-mysql ...
Out 10 11:36:59 centos7.dqs.local docker-compose[30104]: [113B blob data]
Out 10 11:36:59 centos7.dqs.local docker-compose[29402]: 
Out 10 11:36:59 centos7.dqs.local systemd[1]: Stopped wordpress service with docker compose.
```

Agora vamos consultar os containers

```bash
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Como podemos notar não temos nenhum container ou seja, quando o docker compose subir novamente ele vai recriar os containers e utilizar os dados que temos armazenados via a configuração de volumes do docker-compose.yml

Agora vamos subir novamente o serviço e testar

```bash
systemctl start docker-compose@wordpress
```

Agora vamos consultar o status do serviço

```bash
systemctl status docker-compose@wordpress -l
● docker-compose@wordpress.service - wordpress service with docker compose
   Loaded: loaded (/etc/systemd/system/docker-compose@.service; enabled; vendor preset: disabled)
   Active: active (running) since Qui 2019-10-10 11:39:01 -03; 8s ago
  Process: 30104 ExecStop=/usr/bin/docker-compose down (code=exited, status=0/SUCCESS)
 Main PID: 30258 (docker-compose)
   CGroup: /system.slice/system-docker\x2dcompose.slice/docker-compose@wordpress.service
           ├─30258 /usr/bin/docker-compose up
           └─30259 /usr/bin/docker-compose up

Out 10 11:39:03 centos7.dqs.local docker-compose[30258]: wp-mysql | 2019-10-10T14:39:03.741574Z 0 [Note]   - '::' resolves to '::';
Out 10 11:39:03 centos7.dqs.local docker-compose[30258]: wp-mysql | 2019-10-10T14:39:03.741598Z 0 [Note] Server socket created on IP: '::'.
Out 10 11:39:03 centos7.dqs.local docker-compose[30258]: wp-mysql | 2019-10-10T14:39:03.743589Z 0 [Warning] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
Out 10 11:39:03 centos7.dqs.local docker-compose[30258]: wp-mysql | 2019-10-10T14:39:03.755647Z 0 [Note] Event Scheduler: Loaded 0 events
Out 10 11:39:03 centos7.dqs.local docker-compose[30258]: wp-mysql | 2019-10-10T14:39:03.755834Z 0 [Note] mysqld: ready for connections.
Out 10 11:39:03 centos7.dqs.local docker-compose[30258]: wp-mysql | Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
Out 10 11:39:04 centos7.dqs.local docker-compose[30258]: wp-web | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 192.168.176.3. Set the 'ServerName' directive globally to suppress this message
Out 10 11:39:04 centos7.dqs.local docker-compose[30258]: wp-web | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 192.168.176.3. Set the 'ServerName' directive globally to suppress this message
Out 10 11:39:04 centos7.dqs.local docker-compose[30258]: wp-web | [Thu Oct 10 14:39:04.384978 2019] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.38 (Debian) PHP/7.3.10 configured -- resuming normal operations
Out 10 11:39:04 centos7.dqs.local docker-compose[30258]: wp-web | [Thu Oct 10 14:39:04.385066 2019] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
```

Agora vamos consultar os containers

```bash
docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
0ed73e309e33        wordpress           "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        0.0.0.0:80->80/tcp                  wp-web
a6bf2323c534        mysql:5.7           "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp   wp-mysql
```

Agora vamos consultar o nosso Wordpress novamente em: http://10.3.0.91

Como podemos notar o nosso serviço está funcionando corretamente e temos os dados persistidos via volumes.

## Criando mais serviços para o Systemd

Agora parar criarmos um serviço para, precisamos começar criando o diretório para armazenar o nosso novo serviço.

```bash
mkdir -p /srv/docker/compose/drupal
```

Agora crie o seu arquivo docker-compose.yml dentro do diretório /srv/docker/compose/drupal

```yaml
vim /srv/docker/compose/drupal/docker-compose.yml
version: '3'
services:
  drupal:
    image: drupal:8-apache
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-themes:/var/www/html/themes
      - drupal-sites:/var/www/html/sites
    networks:
      - net
    restart: always
  postgres:
    image: postgres:10
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - net
    restart: always
volumes:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:
  db_data:
networks:
  net:
```

Para habilitar o novo serviço

```bash
systemctl enable docker-compose@drupal
```

Agora basta iniciar o novo serviço

```bash
systemctl start docker-compose@drupal
```

Agora já podemos configurar o Drupal em http://10.3.0.91:8080

## Cleanup para o Docker

Agora vamos configurar um cleanup para o Docker, em alguns casos temos informações temporárias que não seram mais utilizadas, desta forma é bom manter sempre o servidor limpo.

Caso executemos o docker prune manualmente vamos ter

```bash
docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

Are you sure you want to continue? [y/N] y
Deleted Networks:
test_network
docker-compose_default
prestashop_my_net
new_network
net1
net2
docker-compose_test_net

Total reclaimed space: 0B
```

**Caso não queira remover os arquivos não configure este serviço.**

Vamos criar o arquivo de controle do serviço

```bash
vim /etc/systemd/system/docker-cleanup.timer

[Unit]
Description=Docker cleanup timer

[Timer]
OnUnitInactiveSec=12h

[Install]
WantedBy=timers.target
```

Agora vamos criar o serviço 

```bash
vim /etc/systemd/system/docker-cleanup.service
[Unit]
Description=Docker cleanup
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
WorkingDirectory=/tmp
User=root
Group=root
ExecStart=/usr/bin/docker system prune -f

[Install]
WantedBy=multi-user.target
```

Agora vamos habilitar o serviço

```bash
systemctl enable docker-cleanup.timer
```

## Referências

- https://gist.github.com/mosquito/b23e1c1e5723a7fd9e6568e5cf91180f
