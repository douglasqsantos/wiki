# MySQL com Docker e Docker Compose no Systemd

Galera aqui vamos brincar um pouco com o MySQL no Docker, vou demostrar a utilização básica do container e vamos implementando as funcionalidades para deixar o container do MySQL como produção.

Nota: Caso você ainda não tenha o Docker ou o Docker Compose instalado siga o procedimento das distribuições com suporte em: [Instalação do Docker](https://github.com/douglasqsantos/wiki#docker-installation-instala%C3%A7%C3%A3o-do-docker)

Após o Docker e o Docker Compose Instalados já podemos começar os nossos testes.

Vamos subir um container simples do MySQL informações dos parâmetros que podemos passar para a imagem podem ser consultados em: [MySQL Docker Hub](https://hub.docker.com/_/mysql)

```bash
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=1234 mysql:5.7
```

Explicando o comando acima:

* **docker**: Comando que conversa com a Docker API
* **run**: parâmetro executa um comando em um container
* **-d**: roda um container em Background
* **-p**: Mapea a porta do container para o host. Caso não seja mapeada uma porta não temos como acessar externamente a porta.
* **3306:3306**: a porta 3306 do host vai apontar para a porta 3306 do container
* **--name**: define um nome para o container é uma boa prática caso contrario o Docker vai atribuir um nome aleatório.
* **-e**: Seta variáveis de ambiente para o MySQL no caso a senha do root
* **mysql:5.7**: Imagem do MySQL que vamos utilizar e o 5.7 é a tag que identifica a versão do MySQL

Agora vamos listar os containers para certificar que o nosso esta rodando

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                               NAMES
6add5e53a680        mysql:5.7           "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
```

Como podemos notar o nosso container esta funcionando, agora vamos acessar o serviço do MySQL.

Precisamos instalar algum cliente do MySQL ou MariaDB para conectarmos.

Para Ubuntu podemos instalar

```bash
sudo apt-get install mariadb-client -y
```

Para Debian podemos instalar

```bash
sudo apt-get install mysql-client -y
```

Para Fedora podemos instalar

```bash
sudo dnf install mariadb -y
```

Para CentOS podemos instalar

```bash
sudo yum install mariadb -y
```

Vamos conectar no MySQL com o usuário root e senha que definimos na criação do container 1234

```bash
mysql -u root -p -h 192.168.25.70
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```

Agora vamos criar um banco de dados para teste

```bash
MySQL [(none)]> create database DevOps;
Query OK, 1 row affected (0.001 sec)
```

Agora vamos listar os nossos bancos de dados do MySQL

```bash
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DevOps             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.005 sec)

MySQL [(none)]> \q
Bye
```

Agora vamos parar o nosso container

```bash
docker stop mysql
```

Agora vamos listar os containers

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Como podemos notar o container não esta em execução desta forma pra podermos visualizar ele precisamos passar a flag **-a**

```bash
docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS               NAMES
6add5e53a680        mysql:5.7           "docker-entrypoint.s…"   14 minutes ago      Exited (0) About a minute ago                       mysql
```

Vamos subir o nosso container novamente

```bash
docker start mysql
```

Vamos listar os containers novamente

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
6add5e53a680        mysql:5.7           "docker-entrypoint.s…"   14 minutes ago      Up 20 seconds       0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
```

Vamos acessar novamente o MySQL

```bash
mysql -u root -p -h 192.168.25.70
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DevOps             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.010 sec)

MySQL [(none)]> \q
Bye
```

Como podemos notar o nosso banco de dados ainda existe. Vamos agora remover o container e recriar ele.

```bash
docker rm -f mysql
```

Agora vamos listar todos os containers com a flag de **-a**

```bash
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Agora vamos recriar o nosso container com os mesmos parâmetros.

```bash
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=1234 mysql:5.7
```

Agora vamos acessar novamente o container e vamos listar os bancos de dados

```bash
mysql -u root -p -h 192.168.25.70
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.002 sec)

MySQL [(none)]> \q
Bye
```

Como podemos notar não existe mais o nosso banco de dados.

Nota: Os dados dos containers são armazenados somente na ultima camada da imagem (container layer) desta forma quando excluímos um container todos os dados são descartados.

Vamos criar um diretório para armazenar os dados do nosso banco de dados então

```bash
sudo mkdir -p /srv/mysql/data
```

Agora vamos excluir o container

```bash
docker rm -fv mysql
```

Vamos recriar ele utilizando o diretório /srv/mysql/data como diretório para armazenar os dados.

```bash
docker run -d -p 3306:3306 --name mysql -e "MYSQL_ROOT_PASSWORD=1234" -v /srv/mysql/data:/var/lib/mysql  mysql:5.7
```

Agora estamos passando uma nova flag:

* **-v**: Mapea um volume para o container
* **/srv/mysql/data**: diretório local que vamos mapear para o Container
* **/var/lib/mysql**: diretório que vai receber o mapeamento dentro do container

Agora vamos listar os logs do MySQL

```bash
docker logs mysql
[...]
2019-10-13T20:31:59.074640Z 0 [Note] InnoDB: 96 redo rollback segment(s) found. 96 redo rollback segment(s) are active.
2019-10-13T20:31:59.074708Z 0 [Note] InnoDB: 32 non-redo rollback segment(s) are active.
2019-10-13T20:31:59.075597Z 0 [Note] InnoDB: Waiting for purge to start
2019-10-13T20:31:59.126027Z 0 [Note] InnoDB: 5.7.27 started; log sequence number 12434349
2019-10-13T20:31:59.126795Z 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2019-10-13T20:31:59.126829Z 0 [Note] Plugin 'FEDERATED' is disabled.
2019-10-13T20:31:59.142556Z 0 [Note] Found ca.pem, server-cert.pem and server-key.pem in data directory. Trying to enable SSL support using them.
2019-10-13T20:31:59.143691Z 0 [Warning] CA certificate ca.pem is self signed.
2019-10-13T20:31:59.148167Z 0 [Note] InnoDB: Buffer pool(s) load completed at 191013 20:31:59
2019-10-13T20:31:59.154108Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
2019-10-13T20:31:59.155726Z 0 [Note] IPv6 is available.
2019-10-13T20:31:59.155780Z 0 [Note]   - '::' resolves to '::';
2019-10-13T20:31:59.155836Z 0 [Note] Server socket created on IP: '::'.
2019-10-13T20:31:59.171031Z 0 [Warning] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2019-10-13T20:31:59.198589Z 0 [Note] Event Scheduler: Loaded 0 events
2019-10-13T20:31:59.199366Z 0 [Note] mysqld: ready for connections.
Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```

Como podemos notar o nosso container esta funcionando, vamos listar ele

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                 NAMES
4e7b69a62e38        mysql:5.7           "docker-entrypoint.s…"   About a minute ago   Up About a minute   3306/tcp, 33060/tcp   mysql
```

Agora vamos listar o conteúdo do diretório que mapeamos para o MySQL

```bash
sudo ls -l /srv/mysql/data/
total 188476
-rw-r----- 1 systemd-coredump systemd-coredump       56 Oct 13 20:31 auto.cnf
-rw------- 1 systemd-coredump systemd-coredump     1679 Oct 13 20:31 ca-key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1107 Oct 13 20:31 ca.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1107 Oct 13 20:31 client-cert.pem
-rw------- 1 systemd-coredump systemd-coredump     1679 Oct 13 20:31 client-key.pem
-rw-r----- 1 systemd-coredump systemd-coredump     1346 Oct 13 20:31 ib_buffer_pool
-rw-r----- 1 systemd-coredump systemd-coredump 50331648 Oct 13 20:32 ib_logfile0
-rw-r----- 1 systemd-coredump systemd-coredump 50331648 Oct 13 20:31 ib_logfile1
-rw-r----- 1 systemd-coredump systemd-coredump 79691776 Oct 13 20:32 ibdata1
-rw-r----- 1 systemd-coredump systemd-coredump 12582912 Oct 13 20:32 ibtmp1
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Oct 13 20:31 mysql
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Oct 13 20:31 performance_schema
-rw------- 1 systemd-coredump systemd-coredump     1675 Oct 13 20:31 private_key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump      451 Oct 13 20:31 public_key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1107 Oct 13 20:31 server-cert.pem
-rw------- 1 systemd-coredump systemd-coredump     1675 Oct 13 20:31 server-key.pem
drwxr-x--- 2 systemd-coredump systemd-coredump    12288 Oct 13 20:31 sys
```

Agora vamos logar no MySQL novamente e vamos criar um banco de dados

```bash
mysql -u root -p -h 192.168.25.70
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> CREATE DATABASE DevOps;
Query OK, 1 row affected (0.001 sec)

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DevOps             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.002 sec)

MySQL [(none)]> \q
Bye
```

Agora vamos remover o nosso container

```bash
docker rm -fv mysql
```

Vamos recriar ele utilizando o mesmo diretório /srv/mysql/data que utilizamos para o outro container armazenar os dados.

```bash
docker run -d -p 3306:3306 --name mysql -e "MYSQL_ROOT_PASSWORD=1234" -v /srv/mysql/data:/var/lib/mysql  mysql:5.7
```

Como podemos notar o nosso container esta funcionando, vamos listar ele

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
ba18ef697aed        mysql:5.7           "docker-entrypoint.s…"   45 seconds ago      Up 44 seconds       0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
```

Note que o id anterior do nosso container era: **4e7b69a62e38** agora temos o container com o id: **ba18ef697aed**

Agora vamos logar no MySQL e validar os dados

```bash
mysql -u root -p -h 192.168.25.70
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DevOps             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.002 sec)

MySQL [(none)]> \q
Bye
```

Como podemos notar agora temos o nosso banco de dados mesmo excluindo o container.

**Nota:** Caso perca o container e precise reutilizar os dados que estão no diretório de dados utilize a mesma versão da imagem para não ter problemas de corromper os dados.

## Criando o Docker Compose para o MySQL

Docker compose é uma ferramenta para rodar aplicações Multi-Container. Com um arquivo Docker compose, você pode configurar os serviços necessários para a sua aplicação. Então com um único comando você pode iniciar todos os serviços necessários para a sua aplicação rodar.

Vamos criar um diretório para armazenar o nosso docker-compose.yml

```bash
sudo mkdir -p /srv/docker/compose/mysql
```

Agora vamos criar o nosso docker-compose.yml

```yaml
sudo vim /srv/docker/compose/mysql/docker-compose.yml
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    ports: 
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 1234
    volumes:
      - "/srv/mysql/data:/var/lib/mysql"
```

Explicando o arquivo docker-compose.yml do MySQL:

* **version**: Versão do docker-composer.yml os parâmetros que ele suporta depende da versão.
* **services**: Sessão obrigatório do arquivo abaixo dela que vamos definir os nossos serviços
* **mysql**: Nome do nosso serviço
* **image**: Imagem que vai ser utilizada para subir o container
* **container_name**: Nome do container que vamos subir
* **ports**: Mapeamento das portas do container para a máquina
* **environment**: São as variáveis de ambiente que vamos passar para o container subir
* **volumes**: Mapeamento o diretório de dados para o container.

Agora vamos remover o container que subimos anteriormente

```bash
docker rm -fv mysql
```

Agora vamos acessar o diretório do docker-compose.yml

```bash
cd /srv/docker/compose/mysql/
```

Agora vamos subir o serviço

```bash
docker-compose up -d
Creating network "mysql_default" with the default driver
Creating mysql ... done
```

Agora vamos validar se o nosso container esta funcionando

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
5318ee46479c        mysql:5.7           "docker-entrypoint.s…"   29 seconds ago      Up 27 seconds       0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
```

Vamos listar o nosso container com o docker-compose

```bash
docker-compose ps
Name              Command             State                 Ports
-------------------------------------------------------------------------------
mysql   docker-entrypoint.sh mysqld   Up      0.0.0.0:3306->3306/tcp, 33060/tcp
```

Agora vamos listar os dados do Banco de Dados

```bash
mysql -u root -p -h 192.168.25.70
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DevOps             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.002 sec)

MySQL [(none)]> \q
Bye
```

Agora vamos parar o container do MySQL para configurar o Systemd

```bash
docker-compose down
Stopping mysql ... done
Removing mysql ... done
Removing network mysql_default
```

## Criando o Serviço do Systemd para subir o Container

Agora vamos criar o serviço para manipular o nosso container como um serviço qualquer no SO.

Lembrando que o nosso serviço do **docker-compose.yml** esta em /srv/docker/compose/mysql/docker-compose.yml desta forma o script abaixo esta utilizando a path: **WorkingDirectory=/srv/docker/compose/%i** onde sera trocado o **%i** pelo **mysql**.

Vamos criar o arquivo que vai manipular os serviços do docker-compose

```bash
sudo vim /etc/systemd/system/docker-compose@.service
[Unit]
Description=%i service with docker compose
Requires=docker.service
After=docker.service

[Service]
Restart=always

#Diretório que vai armazenar os arquivos de compose
WorkingDirectory=/srv/docker/compose/%i

# Subindo o Compose
ExecStart=/usr/bin/docker-compose up

# Parando o Compose
ExecStop=/usr/bin/docker-compose down

[Install]
WantedBy=multi-user.target
```

Agora vamos habilitar o serviço do wordpress para o serviço do Docker Compose

```bash
sudo systemctl enable docker-compose@mysql
```

Agora vamos subir o serviço

```bash
sudo systemctl start docker-compose@mysql
```

Agora vamos validar se o serviço está funcionando

```bash
sudo systemctl status docker-compose@mysql -l
● docker-compose@mysql.service - mysql service with docker compose
   Loaded: loaded (/etc/systemd/system/docker-compose@.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2019-10-13 21:21:35 UTC; 56s ago
 Main PID: 5298 (docker-compose)
    Tasks: 4 (limit: 1096)
   Memory: 62.1M
   CGroup: /system.slice/system-docker\x2dcompose.slice/docker-compose@mysql.service
           ├─5298 /usr/bin/docker-compose up
           └─5307 /usr/bin/docker-compose up

Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.592630Z 0 [Warning] CA certificate ca.pem is self signed.
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.595843Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.595994Z 0 [Note] IPv6 is available.
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.596020Z 0 [Note]   - '::' resolves to '::';
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.596052Z 0 [Note] Server socket created on IP: '::'.
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.597473Z 0 [Note] InnoDB: Buffer pool(s) load completed at 191013 21:21:38
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.603982Z 0 [Warning] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is ac
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.620408Z 0 [Note] Event Scheduler: Loaded 0 events
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:21:38.621461Z 0 [Note] mysqld: ready for connections.
Oct 13 21:21:38 ubuntu19 docker-compose[5298]: mysql    | Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```

Agora vamos parar o serviço para podermos testar

```bash
sudo systemctl stop docker-compose@mysql
```

Agora vamos consultar o serviço novamente

```bash
sudo systemctl status docker-compose@mysql
● docker-compose@mysql.service - mysql service with docker compose
   Loaded: loaded (/etc/systemd/system/docker-compose@.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Sun 2019-10-13 21:23:40 UTC; 42s ago
  Process: 5298 ExecStart=/usr/bin/docker-compose up (code=exited, status=0/SUCCESS)
  Process: 5513 ExecStop=/usr/bin/docker-compose down (code=exited, status=0/SUCCESS)
 Main PID: 5298 (code=exited, status=0/SUCCESS)

Oct 13 21:23:39 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:23:39.691330Z 0 [Note] Shutting down plugin 'mysql_native_password'
Oct 13 21:23:39 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:23:39.691592Z 0 [Note] Shutting down plugin 'binlog'
Oct 13 21:23:39 ubuntu19 docker-compose[5298]: mysql    | 2019-10-13T21:23:39.693046Z 0 [Note] mysqld: Shutdown complete
Oct 13 21:23:39 ubuntu19 docker-compose[5298]: mysql    |
Oct 13 21:23:40 ubuntu19 docker-compose[5298]: mysql exited with code 0
Oct 13 21:23:40 ubuntu19 docker-compose[5513]: [55B blob data]
Oct 13 21:23:40 ubuntu19 docker-compose[5513]: [67B blob data]
Oct 13 21:23:40 ubuntu19 docker-compose[5298]:
Oct 13 21:23:40 ubuntu19 systemd[1]: docker-compose@mysql.service: Succeeded.
Oct 13 21:23:40 ubuntu19 systemd[1]: Stopped mysql service with docker compose.
```

Vamos subir o serviço novamente

```bash
sudo systemctl start docker-compose@mysql
```

Agora vamos testar o acesso ao MySQL

```bash
mysql -u root -p -h 192.168.25.70
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DevOps             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.002 sec)

MySQL [(none)]> \q
Bye
```

Agora vamos acesso normalmente ao MySQL e temos o nosso banco de dados.

Agora vamos fazer um último test reiniciando o servidor

```bash
sudo reboot
```

Após logar novamente no servidor vamos validar o nosso serviço do MySQL que esta rodando no container

```bash
sudo systemctl status docker-compose@mysql -l
● docker-compose@mysql.service - mysql service with docker compose
   Loaded: loaded (/etc/systemd/system/docker-compose@.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2019-10-13 21:26:37 UTC; 37s ago
 Main PID: 998 (docker-compose)
    Tasks: 4 (limit: 1096)
   Memory: 81.4M
   CGroup: /system.slice/system-docker\x2dcompose.slice/docker-compose@mysql.service
           ├─ 998 /usr/bin/docker-compose up
           └─1030 /usr/bin/docker-compose up

Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.646926Z 0 [Note] Found ca.pem, server-cert.pem and server-key.pem in data directory. Trying to enable SS
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.650107Z 0 [Warning] CA certificate ca.pem is self signed.
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.658178Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.660168Z 0 [Note] IPv6 is available.
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.662188Z 0 [Note]   - '::' resolves to '::';
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.663550Z 0 [Note] Server socket created on IP: '::'.
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.677582Z 0 [Warning] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is acc
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.792028Z 0 [Note] Event Scheduler: Loaded 0 events
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | 2019-10-13T21:26:43.793887Z 0 [Note] mysqld: ready for connections.
Oct 13 21:26:43 ubuntu19 docker-compose[998]: mysql    | Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```

Como podemos notar tudo funcionando como um serviço instalado localmente

## Referências

* https://docs.docker.com/reference/
* https://github.com/docker/compose