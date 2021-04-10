# Script para instalação do Docker e Docker Compose

Galera, o procedimento da instalação do Docker e Docker Compose pode ser chato em muitas vezes e podemos errar na instalação ou remoção de pacotes, desta forma eu criei um script para a instalação do Docker e do Docker Compose.

**Sistemas suportados:**

- **Ubuntu**: >= 16.04
- **Debian**: >= 9
- **Centos**: 7
- **Fedora**: >= 28

O script se encontra no GitHub desta forma caso o Docker suporte outras versões dos sistemas operacionais eu irei atualizar o script

Para efetuar a instalação basta efetuar o download do script

```bash
curl -L https://raw.githubusercontent.com/douglasqsantos/DevOps/master/Docker/install-docker.sh -o install-docker.sh
```

Agora precisa somente executar

```bash
sudo bash install-docker.sh
```

No final da instalação é solicitado que seja inserido o usuário comum no grupo do docker para manter as boas práticas e não utilizar o usuário root.

```bash
usermod -aG docker douglas
```

## Referências

- [DQS Docker GitHub](https://github.com/douglasqsantos/DevOps/blob/master/Docker)
