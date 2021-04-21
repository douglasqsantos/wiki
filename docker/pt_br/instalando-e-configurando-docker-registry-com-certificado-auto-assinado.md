# Instalando e Configurando Docker Registry com Certificado Auto-Assinado.

Aqui eu vou fazer a Instalação e configuração do Docker Registry que é um respositório local para imagens que vão ser utilizadas pelos clientes Docker.

Nós temos a opção de usar o https://hub.docker.com ou https://github.com/docker-library/docs com várias opções de imagens prontas para uso, porém em vários casos precisamos ter um repostório local dentro da empresa com as configurações e ajustes pertinentes a nossa empresa com isso não é uma boa ideia deixar isso virado para a internet.

Vamos ao trabalho, eu vou levar em consideração que você já tem o docker instalado, caso não tenha siga: https://docs.docker.com/engine/installation/

Pre-requisitos:

- Configuração do dns com o endereço do register caso não tenha um servidor de dns configurado tem muitos passo-a-passo pelo a internet de como configurar, **inserir o nome do host no /etc/hosts não funciona!!!!**
- Nome da máquina: registry.douglasqsantos.com.br
- IP: 192.168.25.111

Como em muitos casos não vamos adquirir um certificado válido para as operações do docker precisamos criar um certificado auto assinado.

Vamos criar o diretório que vai armazenar os certificados.

```bash
mkdir -p certs
```

Agora vamos gerar o nosso certificado auto-assinado

**Aviso:** O CN do certificado tem que ser o nome da maquina ex: registry.douglasqsantos.com.br

```bash
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
  -x509 -days 365 -out certs/domain.crt
Generating a 4096 bit RSA private key
....................................................................................................................................................................................................................................................++
......++
writing new private key to 'certs/domain.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:BR
State or Province Name (full name) [Some-State]:Parana
Locality Name (eg, city) []:Curitiba
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Douglas
Organizational Unit Name (eg, section) []:IT
Common Name (e.g. server FQDN or YOUR name) []:registry.douglasqsantos.com.br
Email Address []:douglas.q.santos.com.br
```

Agora precisamos criar o diretório que vai conter o domain.crt que vai trabalhar como CA.

```bash
mkdir -p /etc/docker/certs.d/registry.douglasqsantos.com.br:5000
```

Agora vamos copiar o domain.crt para o diretório que acabamos de criar para que possamos enviar e obter imagens do Registry.

```bash
cp ./certs/domain.crt /etc/docker/certs.d/registry.douglasqsantos.com.br:5000/ca.crt
```

Agora precisamos reiniciar o serviço do docker.

Debian Jessie/CentOS 7

```bash
systemctl restart docker
```

Padrão SystemV

```bash
/etc/init.d/docker restart
```

Ou em alguns casos

```bash
service docker restart
```

Agora já podemos subir o servior de Registry

```bash
docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
Unable to find image 'registry:2' locally
2: Pulling from library/registry

d52e2f2a2cb9: Pull complete 
b74310c7215e: Pull complete 
d81ab5b97628: Pull complete 
c53aff2e8ca6: Pull complete 
1d4772feffd1: Pull complete 
05c9677bead7: Pull complete 
4ec2e28c3a7b: Pull complete 
Digest: sha256:eac8e342190d22fb1220a1198c62db8d0867cdb237b7eeaca3d1a37090ddc152
Status: Downloaded newer image for registry:2
19d588f3d39da868795e253286ef642265842d509390b00fbc4fd6f2fbd8dd49
```

Vamos listar as nossas imagens

```bash
docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
registry                        2                   4ec2e28c3a7b        13 days ago         224.5 MB
```

Vamos listar os containers que estão rodando

```bash
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
19d588f3d39d        registry:2          "/bin/registry /etc/d"   59 seconds ago      Up 59 seconds       0.0.0.0:5000->5000/tcp   registry
```

Como podemos notar o Registry está rodando em 0.0.0.0:5000 ou sejá vamos precisar utiliar o nome da maquina + a porta 5000 ex: registry.douglasqsantos.com.br:5000

Agora vamos obter uma imagem para testes

```bash
docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
Digest: sha256:457b05828bdb5dcc044d93d042863fba3f2158ae249a6db5ae3934307c757c54
Status: Downloaded newer image for ubuntu:latest
```

Agora vamos listar as imagens para confirmar que a imagem existe

```bash
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
registry            2                   4ec2e28c3a7b        13 days ago         224.5 MB
ubuntu              latest              6cc0fc2a5ee3        13 days ago         187.9 MB
```

Agora precisamos criar uma tag na imagem

```bash
docker tag ubuntu registry.douglasqsantos.com.br:5000/ubuntu
```

Agora vamos enviar a imagem para o Registry

```bash
docker push registry.douglasqsantos.com.br:5000/ubuntu
The push refers to a repository [registry.douglasqsantos.com.br:5000/ubuntu] (len: 1)
6cc0fc2a5ee3: Pushed 
f80999a1f330: Pushed 
2ef91804894a: Pushed 
92ec6d044cb3: Pushed 
latest: digest: sha256:cd0525a6766551ef7f09bc2c74f5c5f701deaaff2cfba1a64fb5aff3cca05a3c size: 6800
```

## Configurando Cliente Debian Jessie

Como o certificado é auto-assinado precisamos configurar o ca.crt para o cliente

```bash
mkdir -p /etc/docker/certs.d/registry.douglasqsantos.com.br:5000
```

Agora precisamos copiar o arquivo do server para o cliente, podemos copiar via scp, rsync.

Ou podemos fazer diferente.

Vamos verificar o certificado no servidor

```bash
cat /etc/docker/certs.d/registry.douglasqsantos.com.br:5000/ca.crt
-----BEGIN CERTIFICATE-----
MIIF+zCCA+OgAwIBAgIJANeGo1glUzWWMA0GCSqGSIb3DQEBCwUAMIGTMQswCQYD
VQQGEwJCUjEPMA0GA1UECAwGUGFyYW5hMREwDwYDVQQHDAhDdXJpdGliYTEQMA4G
A1UECgwHRG91Z2xhczELMAkGA1UECwwCSVQxGTAXBgNVBAMMEGRuczIuZGtzaC5j
b20uYnIxJjAkBgkqhkiG9w0BCQEWF2RvdWdsYXMucS5zYW50b3MuY29tLmJyMB4X
DTE2MDIwMjIyNDIwMVoXDTE3MDIwMTIyNDIwMVowgZMxCzAJBgNVBAYTAkJSMQ8w
DQYDVQQIDAZQYXJhbmExETAPBgNVBAcMCEN1cml0aWJhMRAwDgYDVQQKDAdEb3Vn
bGFzMQswCQYDVQQLDAJJVDEZMBcGA1UEAwwQZG5zMi5ka3NoLmNvbS5icjEmMCQG
CSqGSIb3DQEJARYXZG91Z2xhcy5xLnNhbnRvcy5jb20uYnIwggIiMA0GCSqGSIb3
DQEBAQUAA4ICDwAwggIKAoICAQDUnBViO++gJXgSGYQUXuLvNPER3Vdwd6ecd9dA
+9FZkaYtoAc8LXm4QkVq3gjxAU/y/stDWMC2NZq8W4BrJch4qGq2zdmUSlXw1AWT
6PiRBLV1ht/wCy+bY4lHkOghTe7+s7bTi1HehoOP6uUckW9R8IjU6twEPMK6JRvw
IY2dq2DQitBS2O2+O9/XYVFCBUFhK28sIDsGI+w4gnaz6s2ZsGjqprzxLayH2oEZ
31A8Um1GKivajpx/qWG8LjvQnfHat2U06+z++guWESZ8HOj6qPVULebwXswbNxRo
RFxNxLLR1bH57OuTTqle4AnWMezpWwlgFB53xfrnBQQGTelCbEQMqZ0ME/s8FTOs
xyY/0hovVgcoJhzQVUYvcp0epy4QKPlu6twhz4TYAOmBTmPODdZCox55hISqurH8
eoi9lCJbH+8oWeNvAlYs4Z3HTeL9duvS1ITVGjxxTuEZcAsdLn1O9UpSpPub3Stn
Kf+Al/eFXqb8mDByEGnWwiAFV2z592mTX3SROsumsvuTRIs2OFYJsTzCn5Umg8zJ
KdNJa5euSf7J8IhKfZRx44iMlLoyvlWK5cgX7+nkwTMoxmDHcOU/v3ZjhTTWP/W9
WgNaLl46EyEdw59mTlEG2hUMgIVqpHC4xIjN2Cz2Ck6cpGPu7mROpwWlGV5a0Q8Z
yM6FYQIDAQABo1AwTjAdBgNVHQ4EFgQUDyzVv/gBcO/RCzOWwV1jFwIQ9dEwHwYD
VR0jBBgwFoAUDyzVv/gBcO/RCzOWwV1jFwIQ9dEwDAYDVR0TBAUwAwEB/zANBgkq
hkiG9w0BAQsFAAOCAgEAb1vac7lGtWi526Lz3i4CYKpIDAQbFQWXGb1szfuzaAri
uLRA7MOeC7Vp2VMG5HwF5ugXyAjuUv77hw8kWl2nnoq9bEoj4u00m64Bcy5ztiPC
+Eiswdy7gtFfcJ50TecRkdpPc0jcQjtZgw2C1E9t8B+EO4JAYKrGcdhqFdAUUeh0
j6GxRsFIm6Q8VvT8PRlggbkoIaQz1SbuaacYrC124bNGs9lEf2V4sZ1j+Fm2T0wd
25zt8BDTIt5cEwd9wgImQO2YEHYXVXe9vZtajuhacQOGIkdzrZG21JJaXwDGI+1Q
pOCcKRS8VRqm3TslL3wkLjuetaKxsq/XBNWemo3HdVxMwS57oHd1DLVHgSwnHJK6
Li/TT17+gg+ELb7AwTgwiCrj+ZjiWXI/OuekkT2CpO7BFPQpM8Uflqi+VuVQB0AI
L51UcqKVq3ilNiwznp15FyFX1jwAeskJR9fuqnrjl4GaRinuwtfJPpwjdsrS/s4M
evys89HEU4JJc0h4IvKxUOR6biP0mAWBMdSzeAK0Q9RrfVM1kmfm85Yxp1s8mhXE
W+aUO4fuV2kZQaxnekXDb5+yLy+mwohfIv7Q7b3HX050KCq91zM7Qnz9IoR4jmik
sEblvExWqb/QDaqRSwm30MLES3Y3ynMTfbFsP8NEUrWqt1+mkiUsyXeMiDcuryo=
-----END CERTIFICATE-----
```

O certificado é este texto codificado, com isso podemos copiar este texto é criar um arquivo no cliente com o mesmo conteúdo e mesmo nome.

```bash
vim /etc/docker/certs.d/registry.douglasqsantos.com.br:5000/ca.crt
-----BEGIN CERTIFICATE-----
MIIF+zCCA+OgAwIBAgIJANeGo1glUzWWMA0GCSqGSIb3DQEBCwUAMIGTMQswCQYD
VQQGEwJCUjEPMA0GA1UECAwGUGFyYW5hMREwDwYDVQQHDAhDdXJpdGliYTEQMA4G
A1UECgwHRG91Z2xhczELMAkGA1UECwwCSVQxGTAXBgNVBAMMEGRuczIuZGtzaC5j
b20uYnIxJjAkBgkqhkiG9w0BCQEWF2RvdWdsYXMucS5zYW50b3MuY29tLmJyMB4X
DTE2MDIwMjIyNDIwMVoXDTE3MDIwMTIyNDIwMVowgZMxCzAJBgNVBAYTAkJSMQ8w
DQYDVQQIDAZQYXJhbmExETAPBgNVBAcMCEN1cml0aWJhMRAwDgYDVQQKDAdEb3Vn
bGFzMQswCQYDVQQLDAJJVDEZMBcGA1UEAwwQZG5zMi5ka3NoLmNvbS5icjEmMCQG
CSqGSIb3DQEJARYXZG91Z2xhcy5xLnNhbnRvcy5jb20uYnIwggIiMA0GCSqGSIb3
DQEBAQUAA4ICDwAwggIKAoICAQDUnBViO++gJXgSGYQUXuLvNPER3Vdwd6ecd9dA
+9FZkaYtoAc8LXm4QkVq3gjxAU/y/stDWMC2NZq8W4BrJch4qGq2zdmUSlXw1AWT
6PiRBLV1ht/wCy+bY4lHkOghTe7+s7bTi1HehoOP6uUckW9R8IjU6twEPMK6JRvw
IY2dq2DQitBS2O2+O9/XYVFCBUFhK28sIDsGI+w4gnaz6s2ZsGjqprzxLayH2oEZ
31A8Um1GKivajpx/qWG8LjvQnfHat2U06+z++guWESZ8HOj6qPVULebwXswbNxRo
RFxNxLLR1bH57OuTTqle4AnWMezpWwlgFB53xfrnBQQGTelCbEQMqZ0ME/s8FTOs
xyY/0hovVgcoJhzQVUYvcp0epy4QKPlu6twhz4TYAOmBTmPODdZCox55hISqurH8
eoi9lCJbH+8oWeNvAlYs4Z3HTeL9duvS1ITVGjxxTuEZcAsdLn1O9UpSpPub3Stn
Kf+Al/eFXqb8mDByEGnWwiAFV2z592mTX3SROsumsvuTRIs2OFYJsTzCn5Umg8zJ
KdNJa5euSf7J8IhKfZRx44iMlLoyvlWK5cgX7+nkwTMoxmDHcOU/v3ZjhTTWP/W9
WgNaLl46EyEdw59mTlEG2hUMgIVqpHC4xIjN2Cz2Ck6cpGPu7mROpwWlGV5a0Q8Z
yM6FYQIDAQABo1AwTjAdBgNVHQ4EFgQUDyzVv/gBcO/RCzOWwV1jFwIQ9dEwHwYD
VR0jBBgwFoAUDyzVv/gBcO/RCzOWwV1jFwIQ9dEwDAYDVR0TBAUwAwEB/zANBgkq
hkiG9w0BAQsFAAOCAgEAb1vac7lGtWi526Lz3i4CYKpIDAQbFQWXGb1szfuzaAri
uLRA7MOeC7Vp2VMG5HwF5ugXyAjuUv77hw8kWl2nnoq9bEoj4u00m64Bcy5ztiPC
+Eiswdy7gtFfcJ50TecRkdpPc0jcQjtZgw2C1E9t8B+EO4JAYKrGcdhqFdAUUeh0
j6GxRsFIm6Q8VvT8PRlggbkoIaQz1SbuaacYrC124bNGs9lEf2V4sZ1j+Fm2T0wd
25zt8BDTIt5cEwd9wgImQO2YEHYXVXe9vZtajuhacQOGIkdzrZG21JJaXwDGI+1Q
pOCcKRS8VRqm3TslL3wkLjuetaKxsq/XBNWemo3HdVxMwS57oHd1DLVHgSwnHJK6
Li/TT17+gg+ELb7AwTgwiCrj+ZjiWXI/OuekkT2CpO7BFPQpM8Uflqi+VuVQB0AI
L51UcqKVq3ilNiwznp15FyFX1jwAeskJR9fuqnrjl4GaRinuwtfJPpwjdsrS/s4M
evys89HEU4JJc0h4IvKxUOR6biP0mAWBMdSzeAK0Q9RrfVM1kmfm85Yxp1s8mhXE
W+aUO4fuV2kZQaxnekXDb5+yLy+mwohfIv7Q7b3HX050KCq91zM7Qnz9IoR4jmik
sEblvExWqb/QDaqRSwm30MLES3Y3ynMTfbFsP8NEUrWqt1+mkiUsyXeMiDcuryo=
-----END CERTIFICATE-----
```

Agora precisamos reiniciar o serviço do docker no cliente

```bash
systemctl restart docker
```

Agora vamos obter a imagem que fizemos upload para o Register.

No cliente

```bash
docker pull registry.douglasqsantos.com.br:5000/ubuntu
Using default tag: latest
latest: Pulling from ubuntu

Digest: sha256:cd0525a6766551ef7f09bc2c74f5c5f701deaaff2cfba1a64fb5aff3cca05a3c
Status: Downloaded newer image for registry.douglasqsantos.com.br:5000/ubuntu
```

Algo que podemos notar aqui é que a imagem não precisou ser baixada do Register pois já possuimos a imagem do Ubuntu que são as mesmas layers do repositório com isso não precisamos obter o arquivo.

## Configurando Cliente MAC OS X

A configuração do cliente no MAC OS X é um pouco diferente pois ele utiliza uma VM no VirtualBox para gerenciar o docker.

No MAC OS X inicialize o docker se ele não estiver ativo ele se chama "Docker Quickstart Terminal".

Agora vamos listar as VMs do docker

```bash
docker-machine ls
NAME      ACTIVE   URL          STATE     URL                         SWARM   DOCKER   ERRORS
default   *        virtualbox   Running   tcp://192.168.99.100:2376           v1.9.1   
```

Após iniciar ele precisamos logar na VM via ssh.

```bash
docker-machine ssh default
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 1.9.1, build master : cef800b - Fri Nov 20 19:33:59 UTC 2015
Docker version 1.9.1, build a34a1d5
docker@default:~$ 
```

Aqui precisamos configurar o DNS para a VM

```bash
sudo vi /etc/resolv.conf
domain douglasqsantos.com.br
search douglasqsantos.com.br
nameserver 192.168.25.111
```

Agora se consultarmos o endereço dns da Registry vamos ter algo como abaixo

```bash
nslookup registry.douglasqsantos.com.br
Server:    192.168.25.111
Address 1: 192.168.25.111

Name:      registry.douglasqsantos.com.br
Address 1: 192.168.25.111
```

Agora que o dns está ok, precisamos criar o diretório para armazenar o certificado do servidor.

```bash
sudo mkdir -p /etc/docker/certs.d/registry.douglasqsantos.com.br:5000
```

Agora vamos criar o certificado como fizemos para o cliente Debian Jessie

```bash
sudo vi /etc/docker/certs.d/registry.douglasqsantos.com.br:5000/ca.crt
-----BEGIN CERTIFICATE-----
MIIF+zCCA+OgAwIBAgIJANeGo1glUzWWMA0GCSqGSIb3DQEBCwUAMIGTMQswCQYD
VQQGEwJCUjEPMA0GA1UECAwGUGFyYW5hMREwDwYDVQQHDAhDdXJpdGliYTEQMA4G
A1UECgwHRG91Z2xhczELMAkGA1UECwwCSVQxGTAXBgNVBAMMEGRuczIuZGtzaC5j
b20uYnIxJjAkBgkqhkiG9w0BCQEWF2RvdWdsYXMucS5zYW50b3MuY29tLmJyMB4X
DTE2MDIwMjIyNDIwMVoXDTE3MDIwMTIyNDIwMVowgZMxCzAJBgNVBAYTAkJSMQ8w
DQYDVQQIDAZQYXJhbmExETAPBgNVBAcMCEN1cml0aWJhMRAwDgYDVQQKDAdEb3Vn
bGFzMQswCQYDVQQLDAJJVDEZMBcGA1UEAwwQZG5zMi5ka3NoLmNvbS5icjEmMCQG
CSqGSIb3DQEJARYXZG91Z2xhcy5xLnNhbnRvcy5jb20uYnIwggIiMA0GCSqGSIb3
DQEBAQUAA4ICDwAwggIKAoICAQDUnBViO++gJXgSGYQUXuLvNPER3Vdwd6ecd9dA
+9FZkaYtoAc8LXm4QkVq3gjxAU/y/stDWMC2NZq8W4BrJch4qGq2zdmUSlXw1AWT
6PiRBLV1ht/wCy+bY4lHkOghTe7+s7bTi1HehoOP6uUckW9R8IjU6twEPMK6JRvw
IY2dq2DQitBS2O2+O9/XYVFCBUFhK28sIDsGI+w4gnaz6s2ZsGjqprzxLayH2oEZ
31A8Um1GKivajpx/qWG8LjvQnfHat2U06+z++guWESZ8HOj6qPVULebwXswbNxRo
RFxNxLLR1bH57OuTTqle4AnWMezpWwlgFB53xfrnBQQGTelCbEQMqZ0ME/s8FTOs
xyY/0hovVgcoJhzQVUYvcp0epy4QKPlu6twhz4TYAOmBTmPODdZCox55hISqurH8
eoi9lCJbH+8oWeNvAlYs4Z3HTeL9duvS1ITVGjxxTuEZcAsdLn1O9UpSpPub3Stn
Kf+Al/eFXqb8mDByEGnWwiAFV2z592mTX3SROsumsvuTRIs2OFYJsTzCn5Umg8zJ
KdNJa5euSf7J8IhKfZRx44iMlLoyvlWK5cgX7+nkwTMoxmDHcOU/v3ZjhTTWP/W9
WgNaLl46EyEdw59mTlEG2hUMgIVqpHC4xIjN2Cz2Ck6cpGPu7mROpwWlGV5a0Q8Z
yM6FYQIDAQABo1AwTjAdBgNVHQ4EFgQUDyzVv/gBcO/RCzOWwV1jFwIQ9dEwHwYD
VR0jBBgwFoAUDyzVv/gBcO/RCzOWwV1jFwIQ9dEwDAYDVR0TBAUwAwEB/zANBgkq
hkiG9w0BAQsFAAOCAgEAb1vac7lGtWi526Lz3i4CYKpIDAQbFQWXGb1szfuzaAri
uLRA7MOeC7Vp2VMG5HwF5ugXyAjuUv77hw8kWl2nnoq9bEoj4u00m64Bcy5ztiPC
+Eiswdy7gtFfcJ50TecRkdpPc0jcQjtZgw2C1E9t8B+EO4JAYKrGcdhqFdAUUeh0
j6GxRsFIm6Q8VvT8PRlggbkoIaQz1SbuaacYrC124bNGs9lEf2V4sZ1j+Fm2T0wd
25zt8BDTIt5cEwd9wgImQO2YEHYXVXe9vZtajuhacQOGIkdzrZG21JJaXwDGI+1Q
pOCcKRS8VRqm3TslL3wkLjuetaKxsq/XBNWemo3HdVxMwS57oHd1DLVHgSwnHJK6
Li/TT17+gg+ELb7AwTgwiCrj+ZjiWXI/OuekkT2CpO7BFPQpM8Uflqi+VuVQB0AI
L51UcqKVq3ilNiwznp15FyFX1jwAeskJR9fuqnrjl4GaRinuwtfJPpwjdsrS/s4M
evys89HEU4JJc0h4IvKxUOR6biP0mAWBMdSzeAK0Q9RrfVM1kmfm85Yxp1s8mhXE
W+aUO4fuV2kZQaxnekXDb5+yLy+mwohfIv7Q7b3HX050KCq91zM7Qnz9IoR4jmik
sEblvExWqb/QDaqRSwm30MLES3Y3ynMTfbFsP8NEUrWqt1+mkiUsyXeMiDcuryo=
-----END CERTIFICATE-----
```

Agora precisamos reiniciar o servidor do docker

```bash
sudo /etc/init.d/docker restart
```

Agora podemos sair da VM

```bash
exit
```

Agora podemos obter a imagem do Registry no cliente MAC OS X

```bash
docker pull registry.douglasqsantos.com.br:5000/ubuntu
Using default tag: latest
latest: Pulling from ubuntu

Digest: sha256:cd0525a6766551ef7f09bc2c74f5c5f701deaaff2cfba1a64fb5aff3cca05a3c
Status: Downloaded newer image for registry.douglasqsantos.com.br:5000/ubuntu:latest
```

Aqui tivemos a mesma informação que o Cliente Debian Jessie o cliente já possui a imagem do Ubuntu que são as mesmas layers do repositório com isso não precisamos obter o arquivo.

## Referências

- https://github.com/docker/distribution/blob/master/docs/deploying.md#get-a-certificate
- https://docs.docker.com/registry/insecure/
- https://docs.docker.com/registry/deploying/
