# Docker compartilhando Dados entre containers

Galera aqui eu vou demostrar como compartilhar dados entre containers, vou fazer a utilização da exposição de um Volume via Dockerfile e outra via linha de comando.

O que eu vou utilizar:

- **Container 1**: Flask App
  - **Nome**: web
  - **Porta**: 5000/tcp
  - **Network**: shared_net
  - **Volume compartilhado**: /app/public
- **Container 2**: Redis
  - **Nome**: redis
  - **Porta**: 6379
  - **Network**: shared_net
  - **Volume consumido**: web:/app/public

## Criando a estrutura da Flask App

Vamos utilizar uma aplicação o mais simples possivel somente para utilizamos os recursos de compartilhamento de uma aplicação Real.

Vamos criar o diretório para armazenar os arquivos

```bash
mkdir -p ~/docker_shared/public && cd ~/docker_shared
```

Vamos criar o arquivo de init da app

```bash
touch __init__.py
```

Agora vamos criar o arquivo principal da App

```python
vim app.py
from flask import Flask
from flask_redis import FlaskRedis

app = Flask(__name__)
app.config['REDIS_URL'] = 'redis://redis:6379/0'

redis = FlaskRedis(app)


@app.route('/')
def counter():
    return str(redis.incr('web_counter'))
```

Agora vamos criar o arquivo de dependencias da nossa App

```bash
vim requirements.txt
Flask==0.12
flask-redis==0.3
```

Agora vamos criar o arquivo que vai ser visualizado entre as aplicações
```css
vim public/main.css
/* CSS goes here. */
```

## Criando o Dockerfile

Agora vamos criar o Dockerfile para a nossa App

Vamos acessar o diretório com os nossos arquivos

```bash
cd ~/docker_shared/
```

Agora vamos criar o Dockerfile

```dockerfile
vim Dockerfile
# Imagem base utilizada
FROM python:2.7-alpine
# Criando o diretório para armazenar a App
RUN mkdir /app
# Definindo o Work directory para o build
WORKDIR /app
# Copiando as dependencias para a imagem
COPY requirements.txt requirements.txt
# Instalando as dependencias da nossa App
RUN pip install -r requirements.txt
# Copia todos os arquivos para dentro da imagem
COPY . .
# Definindo as Labels da imagem
LABEL maintainer="Douglas Quintiliano dos Santos <douglas.q.santos@gmail.com@gmail.com>" version="1.0"
# Volume que vai ser exposto
VOLUME ["/app/public"]
# Comando para subir a app
CMD flask run --host=0.0.0.0 --port=5000
```

## Gerando a imagem

Vamos acessar o diretório com os nossos arquivos

```bash
cd ~/docker_shared/
```

Agora vamos gerar a nossa imagem

```bash
docker build -t web:v1 .
Sending build context to Docker daemon  6.656kB
Step 1/9 : FROM python:2.7-alpine
 ---> df73112425b5
Step 2/9 : RUN mkdir /app
 ---> Using cache
 ---> f8faf8d53f32
Step 3/9 : WORKDIR /app
 ---> Using cache
 ---> b32c8b7718c6
Step 4/9 : COPY requirements.txt requirements.txt
 ---> 503c719a896e
Step 5/9 : RUN pip install -r requirements.txt
 ---> Running in 084c8d24ccab
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that date. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support
Collecting Flask==0.12 (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/0e/e9/37ee66dde483dceefe45bb5e92b387f990d4f097df40c400cf816dcebaa4/Flask-0.12-py2.py3-none-any.whl (82kB)
Collecting flask-redis==0.3 (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/8b/59/e29f607475ca6ae21e30ff5e6ca0f2fd58701879a31ffeff7471ea3865f6/Flask_Redis-0.3.0-py2.py3-none-any.whl
Collecting itsdangerous>=0.21 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting click>=2.0 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl (81kB)
Collecting Jinja2>=2.4 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/65/e0/eb35e762802015cab1ccee04e8a277b03f1d8e53da3ec3106882ec42558b/Jinja2-2.10.3-py2.py3-none-any.whl (125kB)
Collecting Werkzeug>=0.7 (from Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/ce/42/3aeda98f96e85fd26180534d36570e4d18108d62ae36f87694b476b83d6f/Werkzeug-0.16.0-py2.py3-none-any.whl (327kB)
Collecting redis>=2.7.6 (from flask-redis==0.3->-r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/cc/ed/c7447328a3d9fb26961ca4ee877629a9514705b9442d3179456cb860c70f/redis-3.3.10-py2.py3-none-any.whl (66kB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->Flask==0.12->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/b9/2e/64db92e53b86efccfaea71321f597fa2e1b2bd3853d8ce658568f7a13094/MarkupSafe-1.1.1.tar.gz
Building wheels for collected packages: MarkupSafe
  Building wheel for MarkupSafe (setup.py): started
  Building wheel for MarkupSafe (setup.py): finished with status 'done'
  Created wheel for MarkupSafe: filename=MarkupSafe-1.1.1-cp27-none-any.whl size=12631 sha256=0339a5c6b42af305615b91d3f3ae42c288e42d2051de058999d92c0a49cb39db
  Stored in directory: /root/.cache/pip/wheels/f2/aa/04/0edf07a1b8a5f5f1aed7580fffb69ce8972edc16a505916a77
Successfully built MarkupSafe
Installing collected packages: itsdangerous, click, MarkupSafe, Jinja2, Werkzeug, Flask, redis, flask-redis
Successfully installed Flask-0.12 Jinja2-2.10.3 MarkupSafe-1.1.1 Werkzeug-0.16.0 click-7.0 flask-redis-0.3.0 itsdangerous-1.1.0 redis-3.3.10
Removing intermediate container 084c8d24ccab
 ---> 91ab393bd049
Step 6/9 : COPY . .
 ---> 0693c1181c8e
Step 7/9 : LABEL maintainer="Douglas Quintiliano dos Santos <douglas.q.santos@gmail.com@gmail.com>" version="1.0"
 ---> Running in d487bb4661fc
Removing intermediate container d487bb4661fc
 ---> 985b30c21c81
Step 8/9 : VOLUME ["/app/public"]
 ---> Running in 176a298a68d8
Removing intermediate container 176a298a68d8
 ---> 171e1aebd267
Step 9/9 : CMD flask run --host=0.0.0.0 --port=5000
 ---> Running in db5249f9025f
Removing intermediate container db5249f9025f
 ---> 6c23435af9c9
Successfully built 6c23435af9c9
Successfully tagged web:v1
```

## Criando a rede compartilhada

Vamos criar agora a rede compartilhada para os containers poderem conversar via nome

```bash
docker network create -d bridge shared_net
c8609de0443ce236d9b3155e67663ddc9936d921c5ed1a86a96dd731431fb038
```

## Rodando os Containers

Vamos acessar o diretório com os arquivos

```bash
cd ~/docker_shared/
```

Agora vamos subir o container da Flask App

```bash
docker run --rm -itd -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --name web --net shared_net -v $PWD:/app web:v1
48a36ff075a73f9b658dccbb83b3bdb164a97ff29b1f3f2e382b7248df27a0ff
```

Agora vamos subir o container do Redis

```bash
docker container run --rm -itd -p 6379:6379 --name redis --net shared_net -v web_redis:/data --volumes-from web redis:3.2-alpine
3565c04a89b4a1ca57bfe156a46f5a01bce844e613f60912b6aadeb14532fffc
```

Agora vamos listar o conteudo de /app/public do container redis que esta consumindo os volumes da nossa imagem da Flask App

```bash
docker exec -it redis sh -c 'ls -l /app/public/'
total 4
-rw-rw-r--    1 root     root            21 Oct 12 13:57 main.css
```

Como podemos notar temos um arquivo main.css que esta sendo exposto na imagem que geramos via a diretiva: **VOLUME ["/app/public"]** que definimos no Dockerfile

Podemos testar a nossa app em : http://ip_do_docker:5000

## Atualizando o Dockerfile

Agora vamos atualizar o Dockerfile para não expor os volumes via imagem

Vamos acessar o diretório com os nossos arquivos

```bash
cd ~/docker_shared/
```

Agora vamos criar o Dockerfile

```dockerfile
vim Dockerfile
# Imagem base utilizada
FROM python:2.7-alpine
# Criando o diretório para armazenar a App
RUN mkdir /app
# Definindo o Work directory para o build
WORKDIR /app
# Copiando as dependencias para a imagem
COPY requirements.txt requirements.txt
# Instalando as dependencias da nossa App
RUN pip install -r requirements.txt
# Copia todos os arquivos para dentro da imagem
COPY . .
# Definindo as Labels da imagem
LABEL maintainer="Douglas Quintiliano dos Santos <douglas.q.santos@gmail.com@gmail.com>" version="1.0"
# Comando para subir a app
CMD flask run --host=0.0.0.0 --port=5000
```

Agora vamos gerar a nossa imagem atualizada

```bash
docker build -t web:v1 .
Sending build context to Docker daemon  9.216kB
Step 1/8 : FROM python:2.7-alpine
 ---> df73112425b5
Step 2/8 : RUN mkdir /app
 ---> Using cache
 ---> f8faf8d53f32
Step 3/8 : WORKDIR /app
 ---> Using cache
 ---> b32c8b7718c6
Step 4/8 : COPY requirements.txt requirements.txt
 ---> Using cache
 ---> 503c719a896e
Step 5/8 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 91ab393bd049
Step 6/8 : COPY . .
 ---> 7e9ad3575a42
Step 7/8 : LABEL maintainer="Douglas Quintiliano dos Santos <douglas.q.santos@gmail.com@gmail.com>" version="1.0"
 ---> Running in 16b4c8b3a9e9
Removing intermediate container 16b4c8b3a9e9
 ---> 9cb9b3074259
Step 8/8 : CMD flask run --host=0.0.0.0 --port=5000
 ---> Running in b769fbe5df06
Removing intermediate container b769fbe5df06
 ---> 81db7ed1fff0
Successfully built 81db7ed1fff0
Successfully tagged web:v1
```

Agora vamos parar os containers

```bash
docker stop redis
docker stop web
```

Agora vamos subir o container da Flask App com a nova imagem, agora vamos expor o volume /app/public via linha de comando

```bash
docker container run --rm -itd -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --name web --net shared_net -v $PWD:/app -v /app/public web:v1
```

Agora vamos subir o container do Redis novamente

```bash
docker container run --rm -itd -p 6379:6379 --name redis --net shared_net -v web_redis:/data --volumes-from web redis:3.2-alpine
```

Agora vamos listar o conteudo do volume compartilhado

```bash
docker exec -it redis sh -c 'ls -l /app/public/'
total 4
-rw-rw-r--    1 root     root            21 Oct 12 13:57 main.css
```

## Referências

- https://docs.docker.com/reference/
