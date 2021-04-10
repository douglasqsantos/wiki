# Instalando o Docker e Docker Compose no Ubuntu 16.04 até 20.04

* IP do Servidor Debian: 192.168.0.30
* Nome do Servidor: ubuntu2004

Galera aqui eu vou demostrar como efetuar a instalação do Docker e do Docker compose no Ubuntu pode ser utilizado para o Ubuntu 16.04 até a versão do Ubuntu 20.04.

Para melhores praticas de segurança eu vou utilizar um usuário comum para efetuar todo o processo desta forma vamos precisar efetuar a configuração do ambiente para isso.

Vamos criar um usuário comum para o sistema.
OBS: Quando estamos trabalhando com um usuário comum no sistema é recomendado que o uid e gid dele seja o numero 1000 pois quando o container subir com dados compartilhados (volumes) o usuário utilizado geralmente vai ser o 1000.

Caso já tenha um usuário com o id e gid 1000 altere os valores com o seguinte comando.

```bash
usermod -u 2000 usuario_com_id_1000
```

Agora vamos criar o grupo com o gid 1000

```bash
groupadd -g 1000 douglas
```

Agora precisar criar o usuário com o grupo 1000

```bash
useradd -m -s /bin/bash douglas -u 1000 -g 1000
```

Agora vamos definir uma senha para o usuário.

```bash
passwd douglas
```

Vamos ajustar o sudo para que ele não fique solicitando a senha

```bash
visudo
[...]
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

Agora vamos adicionar o nosso usuário no grupo adequada para poder executar os comandos com o sudo

```bash
usermod -aG sudo douglas
```

Agora vamos efetuar o logout do sistema

```bash
exit
```

Agora vamos efetuar o login com o nosso usuário comum

```bash
douglas@192.168.0.30's password:
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-37-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat 20 Jun 2020 02:16:28 PM UTC

  System load:  1.46              Processes:               116
  Usage of /:   9.2% of 48.96GB   Users logged in:         0
  Memory usage: 19%               IPv4 address for enp0s3: 192.168.0.30
  Swap usage:   0%


0 updates can be installed immediately.
0 of these updates are security updates.


Last login: Thu Jun 18 17:47:04 2020
```

Agora vamos se certificar que não temos nenhum pacote antigo do docker instalado no sistema

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

Agora vamos atualizar os repositórios

```bash
sudo apt-get update
```

Agora vamos instalar os pré-requisitos para o Docker

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
```

Adicionando a chave gpg para o repositório do docker

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Agora podemos validar a chave importada

```bash
sudo apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

Agora vamos adicionar o repositório do Docker

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Agora vamos atualizar os repositórios

```bash
sudo apt-get update
```

Agora vamos instalar os pacotes do Docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

Agora vamos subir o serviço do Docker

```bash
sudo systemctl start docker
```

Agora vamos inserir o Docker na inicialização do sistema

```bash
sudo systemctl enable docker
```

Agora vamos adicionar o nosso usuário ao grupo Docker

```bash
sudo usermod -aG docker douglas
```

Agora precisamos efetuar logout

```bash
exit
```

Agora vamos efetuar login novamente no sistema

```bash
douglas@192.168.0.30's password:
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-37-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat 20 Jun 2020 02:20:10 PM UTC

  System load:  1.3                Processes:                122
  Usage of /:   10.2% of 48.96GB   Users logged in:          0
  Memory usage: 28%                IPv4 address for docker0: 172.17.0.1
  Swap usage:   0%                 IPv4 address for enp0s3:  192.168.0.30


0 updates can be installed immediately.
0 of these updates are security updates.


Last login: Sat Jun 20 14:16:29 2020 from 192.168.0.101
```

Agora podemos listar os containers

```bash
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## Instalação do Docker Compose

A Instalação do Docker Compose é simples, vamos obter o binário

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Agora vamos adicionar a permisão de execução ao script

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

Agora vamos criar um link para o nosso docker-compose

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Agora vamos validar a versão do nosso Docker Compose

```bash
docker-compose --version
docker-compose version 1.26.0, build d4451659
```

## Ajustes para o VIM

Vamos efetuar a instalação do VIM

```bash
sudo apt-get install -y vim 
```

Agora vamos obter o novo arquivo de configuração

```bash
curl -L https://raw.githubusercontent.com/douglasqsantos/DevOps/master/Misc/prep-vim.sh | bash
```

## Criando um container para teste

Vamos efetuar a criação de um container para testar o nosso Docker

Vamos criar um diretório para armazenar a configuração das nossas images

```bash
mkdir -p docker-images/apache
```

Agora vamos acessar o diretório para criar o nosso arquivo

```bash
cd docker-images/apache
```

Vamos criar o nosso Dockerfile

```bash
vim Dockerfile
FROM ubuntu
RUN apt-get update && apt-get install apache2 -y
CMD apachectl -DFOREGROUND
```

Agora vamos gerar a nossa imagem

```bash
docker build --tag ubuntu_apache:v1 .
```

Agora vamos subir um container com a nossa imagem, vamos mapear a porta 9090 do Ubuntu para a porta 80 do container.

```bash
docker run -d -p 9090:80 --name ubuntu_apache ubuntu_apache:v1
```

Agora vamos verificar se o container está rodando

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
49b5f3c214df        ubuntu_apache:v1    "/bin/sh -c 'apachec…"   5 seconds ago       Up 3 seconds        0.0.0.0:9090->80/tcp   ubuntu_apache
```

Vamos verificar os logs do nosso container

```bash
docker logs ubuntu_apache
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
```

Agora vamos efetuar um teste acessando o ip do servidor docker na porta 9090 http://192.168.25.70:9090

Para acompanhar o consumo do container podemos utilizar o seguinte comando

```bash
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
49b5f3c214df        ubuntu_apache       0.04%               5.434MiB / 982.9MiB   0.55%               976B / 0B           2.45MB / 0B         57
```

## Referências

* [Docker on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
* [Docker Compose](https://docs.docker.com/compose/install/)
