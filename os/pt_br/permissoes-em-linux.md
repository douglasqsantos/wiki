# Permissões em Linux

As permissões são um dos aspectos mais importantes do Linux (na verdade, de todos os sistemas baseados em Unix). Elas são usadas para vários fins, mas servem principalmente para proteger o sistema e os arquivos dos usuários. Manipular permissões é uma atividade interessante, mas complexa ao mesmo tempo.

Mas tal complexidade não deve ser interpretada como dificuldade e sim como possibilidade de lidar com uma grande variedade de configurações, o que permite criar vários tipos de proteção a arquivos e diretórios.

Como você deve saber, somente o super-usuário (root) tem ações irrestritas no sistema, justamente por ser o usuário responsável pela configuração, administração e manutenção do Linux. Cabe a ele, por exemplo, determinar o que cada usuário pode executar, criar, modificar, etc. 

Naturalmente, a forma usada para especificar o que cada usuário do sistema pode fazer é a determinação de permissões. Sendo assim, neste artigo você verá como configurar permissões de arquivos e diretórios, assim como modificá-las.

Vamos começar a analisar as permissões, vamos pegar o arquivo /etc/passwd como exemplo

```bash
ls -l /etc/passwd
-rw-r--r-- 1 root root 1297 Jul 17 12:49 /etc/passwd
```

Agora vamos ver o que significa -rw-r--r--
|Tipo de arquivo | Permissão de Dono |  Permissão de Grupo | Permissão de Outros|
|---|---|---|---|
| - | r w - | r - - | r- - |

Aqui nós temos o tipo de arquivo que é o primeiro caracter da esquerda podemos ter os seguintes tipo de arquivos:

* **d:** Diretório
* **b:** Arquivo de bloco
* **c:** Arquivo de caractere
* **s:** Socket
* **-:** Arquivo regular

Agora nós temos os outros três campos para analisar que são eles:

* **Usuário dono:** Também chamado de owner, é o proprietário do arquivo/diretório
* **Grupo dono:** É o grupo proprietário do arquivo/diretório. As permissões são dadas, cadastrando usuários no grupo atribuído
* **Outros:** Aplica-se a qualquer outro usuário, que não seja o usuário dono e nem pertença ao grupo dono.

Agora vamos ver para que serve o r w x - :

* **Leitura ( r ):** Permissão para visualizar o conteúdo do arquivo (**r** do inglês **read**)
* **Escrita (w):** Permissão para alterar o conteúdo de um arquivo (**w** do inglês **write**)
* **Execução (x):** Permissão para executar arquivos (**x** do inglês **execution**)
* **None(-):** Nesse caso, nenhuma permissão foi atribuída.

Agora que já sabemos o que são os caracteres vamos analisar o arquivo /etc/passwd novamente

```bash
ls -l /etc/passwd
-rw-r--r-- 1 root root 1297 Jul 17 12:49 /etc/passwd
```

Aqui o usuário dono é o **root** que é a terceira coluna da listagem e o grupo **dono** é o grupo root que é a quarta coluna da listagem

O segundo campo é a quantidade de arquivos que se referenciam a ele neste caso somente 1 ele mesmo não tem hardlinks.

Como sabemos que o usuário root é do dono a permissão dele é r w - ou seja ele tem a permissão de ler e alterar este arquivo mas não executar.

O grupo dono tem a permissão r - - que é somente ler, sem a permissão de escrita ou execução do arquivo.

E qualquer outro usuário não sendo o usuário dono ou o grupo dono tem a permissão r - - que é somente ler, sem a permissão de escrita ou execução do arquivo.

## Comando chmod

Podemos utilizar o comando **chmod**  para alterar permissões de um ou mais arquivos/diretórios.

Agora vamos ver os parâmetros e permissões que podemos utilizar no chmod. Para realizar a tarefa de alterar as permissões, são necessárias algumas letras que simbolizam os parâmetros de alteração de usuários, grupos e outros, e as permissões.

Vamos utilizar a seguinte tabela base
|                     Níveis de permissão               ||
|---|---|
| u | Usuário dono do arquivo (user) |
| g | Grupo do arquivo (group|
| o | Outros usuários que não são donos e não estão cadastrados no grupo (other)|
| a | Todos. usuários, grupos e outros (all)|
|            **Tipos de permissão - Notação textual**             ||
| r | Concede ou remove permissão para leitura (read) |
| w | Concede ou remove permissão para escrita (write|
| x | Concede ou remove permissão para execução (execute)|
|            **Tipos de permissão - Notação octal**                ||
| 0 | Indica sem permissão |
| 1 | Indica permissão de execução |
| 2 | Indica permissão de escrita |
| 3 | Indica a permissão de execução e escrita 2 + 1|
| 4 | Indica a permissão de leitura |
| 5 | Indica a permissão de leitura e execução 4 + 1|
| 6 | Indica a permissão de leitura e escrita 4 + 2 |
| 7 | Indica a permissão de leitura, escrita e execução 4 + 2 + 1 |

**OBS:** Quando alterar as permissões de um diretório, é obrigatório adicionar a permissão **x**. A opção **r** permite leitura, porém a opção **x** permite o acesso, por isso, precisam ser configuradas simultaneamente.

Vamos começar a testar isso então.

Para os exemplos eu vou ficar no diretório /srv

```bash
cd /srv
```

Agora vamos copiar o arquivo /etc/passwd para o /srv

```bash
cp /etc/passwd /srv/
```

Agora vamos listar as permissões do arquivo passwd que esta no /srv

```bash
ls -l /srv/passwd 
-rw-r--r-- 1 root root 1297 Dez 22 13:53 /srv/passwd
```

As permissões ainda continuam as mesmas.

Agora vamos dar a permissão de escrita no passwd para o grupo dono

```bash
chmod g+w /srv/passwd
```

Aqui no exemplo acima estamos adicionando a permissão de escrita no arquivo /srv/passwd para o grupo, quando utilizamos o símbolo de **+** estamos adicionando uma permissão, se utilizamos o símbolo de **-** vamos remover uma permissão.

Agora vamos listar as permissões do arquivo novamente

```bash
ls -l /srv/passwd 
-rw-rw-r-- 1 root root 1297 Dez 22 13:53 /srv/passwd
```

Note que agora temos **- r w - r w - r - -** ou seja o usuário e o grupo tem permissão de leitura e escrita o outros ainda continua com a permissão de somente leitura.

Agora vamos mandar remover a permissão de leitura do outros

```bash
chmod o-r /srv/passwd
```

Agora vamos visualizar as permissões do arquivo novamente

```bash
ls -l /srv/passwd 
-rw-rw---- 1 root root 1297 Dez 22 13:53 /srv/passwd
```

Note que agora temos **- r w - r w - - - -** ou seja o usuário dono e o grupo dono tem a permissão de leitura e escrita neste arquivo e outros não tem permissão alguma.

Agora vamos definir a permissão para usuário dono, grupo dono e outros de leitura e escrita neste arquivo

```bash
chmod u=rx,g=rx,o=rx /srv/passwd 
```

Note que no comando chmod acima utilizamos o símbolo de **=** para determinarmos como devem ficar as permissões.

Agora vamos listar as permissões do arquivo

```bash
ls -l /srv/passwd 
-r-xr-xr-x 1 root root 1297 Dez 22 13:53 /srv/passwd
```


As permissões não são sempre **rwx rwx rwx** e quando não tem alguma das permissões tem que ficar o símbolo de **-** ? Sim mas vamos a mais teste, vamos definirmos as permissões colocando no lugar do **w** o símbolo de **-**

```bash
chmod u=r-x,g=r-x,o=r-x /srv/passwd
```

Agora vamos visualizar as permissões do arquivo novamente

```bash
ls -l /srv/passwd 
-r--r--r-- 1 root root 1297 Dez 22 13:53 /srv/passwd
```

Mais agora o arquivo ficou com as permissões de somente **- r - - r - - r - - **, se lembra que acima eu comentei que o símbolo de **-** remove uma permissão se eu passar então no **chmod u=r-x** ele vai adicionar a opção de **r** porém vai remover a permissão de **x** 

Pronto já dei uma explicada nas permissões modo textual agora vamos ver as minhas favoritas as permissões em modo octal

No modo octal não podemos utilizar por exemplo u-1 que seria remover a permissão de execução do arquivo ou diretório, aqui temos que passar sempre como queremos as nossas permissões.

Exemplo quando utilizamos isso aqui **chmod u=rx,g=rx,o=rx** acima estamos definindo a permissão de leitura e execução para todos no modo octal é muito mais simples note

```bash
chmod 555 /srv/passwd
```

Aqui o 5 significa r - x o segundo 5 significa r - x e o último significa r - x agora se eu listar o arquivo vou ter 

```bash
ls -l /srv/passwd 
-r-xr-xr-x 1 root root 1297 Dez 22 13:53 /srv/passwd
```

Agora pensa que eu preciso tirar todas as permissões deste arquivo.

Em modo textual precisaria fazer o seguinte

```bash
chmod u-rx,g-rx,o-rx /srv/passwd
```

Agora se eu listar o arquivo vou ter o seguinte

```bash
ls -l /srv/passwd 
---------- 1 root root 1297 Dez 22 13:53 /srv/passwd
```

Agora vamos deixar as permissões como antes e vamos remover

```bash
chmod 555 /srv/passwd 
```

Agora vamos listar as permissões novamente

```bash
ls -l /srv/passwd 
-r-xr-xr-x 1 root root 1297 Dez 22 13:53 /srv/passwd
```

Agora vamos remover com o modo octal

```bash
chmod 000 /srv/passwd
```

Agora vamos listar as permissões novamente

```bash
ls -l /srv/passwd 
---------- 1 root root 1297 Dez 22 13:53 /srv/passwd
```

Agora vamos criar uma tabela de correspondência entre modo textual e modo octal
| Números octal |  Permissão correspondente | Modo Textual |
|---|---|---|
| 0 | Nenhuma permissão | (- - -)|
| 1 | Permissão de execução | (- - x)|
| 2 | Permissão de escrita | (- w -)|
| 3 | Permissão de escrita e execução |( - w x )|
| 4 | Permissão de leitura | (r - - )|
| 5 | Permissão de leitura e execução | (r - x)|
| 6 | Permissão de leitura e escrita | (r w -)|
| 7 | Permissão de leitura, escrita e execução | (r w x)|

**Obs:** Sempre devemos utilizar 3 números para especificar as permissões por exemplo 644 o 6 é permissão para o dono do arquivo o 4 é permissão para o grupo e o último 4 é permissão para outros.

Vamos fazer um teste de omissão de número vamos definir a permissão octal como 77

```bash
chmod 77 /srv/passwd
```

Agora vamos listar o arquivo

```bash
ls -l /srv/passwd 
----rwxrwx 1 root root 1297 Dez 22 13:53 /srv/passwd
```

Note que o usuário dono do arquivo não tem permissão, ou seja sempre que omitirmos um número o sistema vai jogar um 0 a esquerda então em nosso caso da permissão foi definida como 077 que é sem permissão para o usuário dono porém o grupo dono tem permissão total e outros também. 

## Permissões especiais

Existem algumas permissões especiais de arquivos/diretórios que oferecem funcionalidades além das simples permissões de acesso.

## Permissão especial SUID

O SUID bit é utilizado em arquivos executáveis quando se deseja que o programa seja executado com os privilégios de seu usuário dono. Isso é útil em situações em que um programa precisa acessar determinado recurso, mas os usuários que utilizam não têm permissão para fazer isso diretamente, como é o caso do comando shutdown, que só pode ser executado pelo usuário root. Embora a utilização do SUID bit seja inquestionável, seu uso deve ser feito com muito cuidado, pois um problema em sua configuração pode ter consequências sérias de segurança (especialmente se o SUID for para o usuário root).

Sintaxe do comando

Com permissão em modo textual

```bash
chmod u+s arquivo
```

Como o SUID é utilizado para usuário então inserimos a permissão especial **s** no usuário que tem o símbolo **u**

Agora se quisermos utilizar o modo octal podemos fazer da seguinte forma

```bash
chmod 4775 arquivo
```

Notem que no modo octal agora temos 4 números, se recordam de que eu tinha dito que sempre que não informamos um número ele é trocado por um número 0, assim sempre que setamos as permissões com 3 números em octal o bit especial esta sempre como zero.

|Bit especial | Bit de usuário | Bit de grupo | Bit de outros|
|---|---|---|---|
| 4 | 7 | 7 | 5 |

Agora vamos listar as permissões do arquivo shutdown

```bash
ls -l /sbin/shutdown 
-rwxr-xr-x 1 root root 23696 Mar 26  2012 /sbin/shutdown
```

Note que ele tem as permissões normais agora vamos adicionar a ele a permissão de SUID bit que é o bit para usuário dono

```bash
chmod u+s /sbin/shutdown
```

Agora vamos listar as permissões novamente

```bash
ls -l /sbin/shutdown 
-rwsr-xr-x 1 root root 23696 Mar 26  2012 /sbin/shutdown
```

Note que agora temos - r w s r - x r - x, ao invés de termos um x na permissão de execução do usuário dono temos um s que significa que temos o SUID habilitado.

Agora vamos tirar a permissão de execução do usuário dono para notarmos a diferença da permissão.

```bash
chmod u-x /sbin/shutdown
```

Agora vamos listar o arquivo novamente

```bash
 ls -l /sbin/shutdown
-rwSr-xr-x 1 root root 23696 Mar 26  2012 /sbin/shutdown
```

O s do SUID somente vai estar em minúsculo se o arquivo já tiver a permissão execução caso ele não tenha o SUID vai aparecer em maiúsculo .

Agora vamos adicionar novamente a permissão de execução para o usuário dono

```bash
chmod u+x /sbin/shutdown
```

Agora vamos listar as permissões novamente

```bash
ls -l /sbin/shutdown 
-rwsr-xr-x 1 root root 23696 Mar 26  2012 /sbin/shutdown
```

Agora vamos fazer um teste com um usuário comum execute o seguinte comando

```bash
/sbin/shutdown -r now
```

Agora o sistema foi reiniciado pois demos a permissão para o usuário comum executar o shutdown como se fosse o usuário root :D

Agora vamos tirar a permissão de SUID do shutdown e vamos tentar executar com o usuário comum

```bash
chmod u-s /sbin/shutdown
```

Agora vamos listar o arquivo

```bash
ls -l /sbin/shutdown 
-rwxr-xr-x 1 root root 23696 Mar 26  2012 /sbin/shutdown
```

Agora vamos fazer um teste com um usuário comum execute o seguinte comando

```bash
/sbin/shutdown -r now
shutdown: you must be root to do that!
Usage:    shutdown [-akrhPHfFnc] [-t sec] time [warning message]
      -a:      use /etc/shutdown.allow
      -k:      don't really shutdown, only warn.
      -r:      reboot after shutdown.
      -h:      halt after shutdown.
      -P:      halt action is to turn off power.
      -H:      halt action is to just halt.
      -f:      do a 'fast' reboot (skip fsck).
      -F:      Force fsck on reboot.
      -n:      do not go through "init" but go down real fast.
      -c:      cancel a running shutdown.
      -t secs: delay between warning and kill signal.
      ** the "time" argument is mandatory! (try "now") **
```

Note que agora obtivemos um erro informando que o shutdown tem que ser executado pelo usuário root.

## Permissão especial SGID

O **SGID** bit tem a mesma função do SUID bit, porém é aplicado ao grupo e a diretórios, ou seja, o diretório é executado com os privilégios do grupo a que pertence.

O **SGID** quando criamos diretórios dentro do diretório que tem o SGID bit ativo todos eles terão como grupo dono o mesmo grupo a quem o diretório pertence ou seja o mesmo grupo dono do diretório pai.

Sintaxe do SGID:

Modo textual

```bash
chmod g+s diretorio
```

Modo octal

```bash
chmod 2775 diretorio
```

Vamos fazer um teste, vamos criar um diretório

```bash
mkdir -p /srv/diretorio
```

Agora vamos listar as permissões do diretório

```bash
ls -l /srv/
total 1
drwxr-xr-x 2 root root 1024 Dez 22 19:41 diretorio
```

Agora vamos adicionar o SGID bit neste diretório

```bash
chmod g+s /srv/diretorio
```

Agora vamos listar as permissões do diretório

```bash
ls -l /srv/
total 1
drwxr-sr-x 2 root root 1024 Dez 22 19:41 diretorio
```

Note que agora temos d r w x r - s r - x, aqui ainda continua a mesma ideia do s minúsculo e S maiúsculo.

Agora vamos criar um diretório dentro deste diretório

```bash
mkdir /srv/diretorio/novo
```

Agora vamos listar as permissões

```bash
ls -l /srv/diretorio/
total 1
drwxr-sr-x 2 root root 1024 Dez 22 19:44 novo
```

Note que o diretório filho já veio com a opção de SGID bit ativada como herança.

## Permissão especial Stick Bit

O **stick bit** é utilizado em diretórios compartilhados entre vários usuários (em combinação com permissões de escrita para eles), e que seja desejável que usuários não acessem os arquivos que foram criados por outros. Em outras palavras, um diretório com **stick bit** ligado permite que qualquer usuário crie arquivos, mas apenas os donos de arquivos e diretórios poderão remover ou alterar esses arquivos. Um exemplo do uso do **stick bit** é no diretório /tmp ou uma área de dados, pública. Ele também pode ser aplicado a arquivos para que, em operações de manipulação nesse arquivo, seus dados fiquem alocados na área de troca(**swap**) no lugar de serem alocados diretamente em memória.

Sintaxe de uso:

Em modo textual

```bash
chmod +t diretorio
```

Em modo octal

```bash
chmod 1777 diretorio
```

Aqui a diferença do bit ao invés de ficar como s vai ser um t e a mesma lógica do s maiúsculo e minúsculo.

Vamos fazer um teste, vamos criar um diretório para teste

```bash
mkdir /srv/publico
```

Agora vamos dar a permissão de stick nesse diretório

```bash
chmod 1777 /srv/publico
```

Por que eu dei a permissão de 1777 ? O primeiro número é pra eu ativar o stick bit, o segundo é a permissão total para o usuário dono, o terceiro é permissão total para o grupo dono e o quarto é permissão total para outros, na lógica tudo que for criado ali pode ser excluído por qualquer um, mas não vai funcionar assim :D

Agora vamos listar as permissões do diretório

```bash
ls -l /srv/
total 2
drwxr-sr-x 3 root root 1024 Dez 22 19:45 diretorio
drwxrwxrwt 2 root root 1024 Dez 22 19:59 publico
```

Note que temos todas as permissões ativas e no símbolo de execução para outros temos um t agora.

Agora vamos criar dois usuário para teste.

Vamos criar o jose

```bash
useradd -m -s /bin/bash -c "Usuario de teste" jose
```

Agora vamos setar uma senha para ele

```bash
passwd jose
Digite a nova senha UNIX:
Redigite a nova senha UNIX:
passwd: senha atualizada com sucesso
```

Agora vamos criar a usuária maria

```bash
useradd -m -s /bin/bash -c "Usuaria de teste" maria
```

Agora vamos setar uma senha para ela

```bash
passwd maria
Digite a nova senha UNIX:
Redigite a nova senha UNIX:
passwd: senha atualizada com sucesso
```

Agora vamos logar com a maria e vamos criar um diretório em /srv/publico com o nome de maria

```bash
mkdir /srv/publico/maria
```

Agora vamos listar os diretório do publico

```bash
ls -l /srv/publico/
total 1
drwxr-xr-x 2 maria maria 1024 Dez 22 20:02 maria
```

Note que as permissões estão para usuário dono maria e grupo dono maria sem permissão de escrita no diretório ou seja o qualquer outro usuário não vai poder remover este diretório.

Agora vamos logar com o jose e vamos criar um diretório em /srv/publico com o nome de jose

```bash
mkdir /srv/publico/jos
```

Agora vamos listar os diretórios do publico

```bash
ls -l /srv/publico/
total 2
drwxr-xr-x 2 jose  jose  1024 Dez 22 20:04 jose
drwxr-xr-x 2 maria maria 1024 Dez 22 20:02 maria
```

Note que as permissões estão para o usuário dono jose e grupo dono jose sem permissão de escrita no diretório ou seja, qualquer outro usuário não vai poder remover este diretório.

Agora vamos fazer um teste, com o usuário jose vamos tentar remover o diretório da maria.

```bash
rm -rf /srv/publico/maria
rm: não foi possível remover "/srv/publico/maria": Operação não permitida
```

Note que não podemos remover.

Agora logue com o root novamente e vamos adicionar a opção de SGID bit nesse diretório publico

```bash
chmod g+s /srv/publico
```

Agora logue com o jose novamente e vamos mandar remover o diretório da maria

```bash
rm -rf /srv/publico/maria
rm: não foi possível remover "/srv/publico/maria": Operação não permitida
```

Note que mesmo com a opção de SGID bit não podemos remover pois o stick bit bloqueia :D

## Comando chown

Agora vamos ver o comando **chown** que podemos utilizar para mudar o usuário dono do arquivo.

Vamos listar o arquivo /etc/passwd para tomarmos como exemplo

```bash
ls -lh /etc/passwd
-rw-r--r-- 1 root root 1,2K Dez 22 20:01 /etc/passwd
```

Agora vamos detalhar o que significam esses dados.

|Tipo do arquivo | Permissões | Número de referências para este arquivo | Tamanho do arquivo |Data e hora da alteração deste arquivo | Nome do arquivo |
|---|---|---|---|---|---|
| - | r w - r - - r - - | 1 | 1,2k | Dez 22 20:01 | /etc/passwd |

Para diretórios é a mesma coisa porém ao invés do tipo do arquivo ser **-** será **d** e o número de referências vai ser ao diretório ou seja a quantidade de diretórios filhos e a quantidade de diretórios que ele é filho.

Agora vamos ver como utilizamos o chown

```bash
chown usuario arquivo ou diretorio
```

Vamos criar um diretório em /srv para testarmos

```bash
mkdir /srv/adm
```

Agora vamos listar o diretório

```bash
ls -l /srv/
total 1
drwxr-xr-x 2 root root 1024 Dez 22 20:30 adm
```

Note que o usuário dono e o grupo dono são o root, o sistema sempre vai utilizar por padrão quem esta logado neste caso o root.

Agora vamos mudar o usuário dono do arquivo para jose

```bash
chown jose /srv/adm/
```

Agora vamos listar as permissões novamente

```bash
ls -l /srv/
total 1
drwxr-xr-x 2 jose root 1024 Dez 22 20:30 adm
```

Agora podemos notar que o usuário dono do diretório adm é o jose e o grupo dono é o usuário root, agora como estamos falando de um diretório podemos passar a opção de **-R** para ele aplicar esta alteração no diretório pai e em todos os arquivos filhos da seguinte forma

```bash
chown -R jose /srv/adm
```

Neste caso todos os diretórios e arquivos contidos no diretório adm receberam como usuário dono deles o usuário jose.

## Comando chgrp

O comando **chgrp** é utilizado para efetuar a mudança do grupo dono de um arquivo ou diretório.

Vamos criar um grupo chamado administracao

```bash
groupadd administracao
```

Agora vamos mudar o grupo dono do diretório adm para ser o administracao

```bash
chgrp administracao /srv/adm
```

Agora vamos listar o diretório adm

```bash
ls -l /srv/
total 1
drwxr-xr-x 2 jose administracao 1024 Dez 22 20:30 adm
```

Note que agora temos o usuário dono sendo o jose e o grupo dono sendo o administracao, podemos utilizar o **-R** da mesma forma que no comando chown para diretório com isso ele vai mudar o grupo dono recursivamente de todos os arquivos e diretórios filhos do diretório adm.

Agora pense que precisamos mudar o usuário dono e o grupo dono de uma só vez :D

Vamos criar um novo grupo agora chamado contabilidade.

```bash
groupadd contabilidade
```

Agora vamos mudar o usuário dono do adm para ser a maria e o grupo dono para ser contabilidade aqui utilizamos o comando chown usuario:grupo

```bash
chown maria:contabilidade /srv/adm
```

Agora vamos listar o diretório novamente

```bash
ls -l /srv/
total 1
drwxr-xr-x 2 maria contabilidade 1024 Dez 22 20:30 adm
```

Pela documentação do comando podemos utilizar também chown usuario.grupo porém ai temos um grande problema quando estamos trabalhando com ambientes corporativos aonde o nome dos usuário é nome.sobrenome já estaríamos complicados.

## Comando umask

Já notaram que a cada vez que criamos um arquivo ou diretório ele já vem com uma permissão pré-determinada ?

Vamos criar um diretório de teste

```bash
mkdir /srv/teste
```

Agora vamos listar as permissões do diretório teste

```bash
ls -l /srv/
total 2
drwxr-xr-x 2 maria contabilidade 1024 Dez 22 20:30 adm
drwxr-xr-x 2 root  root          1024 Dez 22 20:46 teste
```

Note que a permissão padrão do diretório é 755 ou r w x r - x r - x, note que eu não modifiquei a permissão do diretório adm somente o usuário dono e o grupo dono e a permissão é a mesma do diretório teste, agora vamos ver a criação de um arquivo.

```bash
touch /srv/teste.txt /srv/teste2.txt
```

Agora vamos listar as permissões dos arquivos

```bash
ls -l /srv/teste*.txt
-rw-r--r-- 1 root root 0 Dez 22 20:48 /srv/teste2.txt
-rw-r--r-- 1 root root 0 Dez 22 20:48 /srv/teste.txt
```

Note que os dois arquivos foram criado com a mesma permissão 644 ou seja r w - r - - r - -

Mas de onde ele tira isso, então isso ai é baseado na umask que é a permissão padrão do sistema

Vamos listar a umask

```bash
umask
0022
```

Olha só temos um número ali 0022 ta mais para que serve isso, é a base do calculo das permissões padrões.

Quando vamos criar um diretório ele se baseia no número 0777 - o valor da umask ex:

```bash
0777 - 0022 = 0755
```

Opa, qual é a permissão que os diretórios são criados por padrão 755 porém, lembre-se que quando omitimos o quarto número da esquerda que é o bit especial ele fica como 0.

E para arquivos como é o calculo ? Para arquivos ele se baseia no número 0666 - o valor da umask ex:

```bash
0666 - 0022 = 0644
```

Opa, qual é a permissão que os arquivos são criados por padrão 644 porém, lembre-se que quando omitimos o quarto número da esquerda que é o bit especial ele fica como 0.


Então para mudarmos o tipo de permissão padrão dos diretórios e arquivos precisamos mudar o valor da umask. E sempre se lembrando que isso vai ser para ambos diretórios e arquivos, e o valor setado para a umask vai ser subtraído do valor 0777 para permissão de diretório e 0666 para permissão de arquivos.

**OBS:** O sistema não deixar que arquivos sejam criados com permissão de execução por padrão mesmo fazendo alteração na umask.

Vamos fazer um teste vamos mudar a permissão da umask para 0011

```bash
umask 0011
```

Como que vai ser o calculo de diretório

```bash
0777 - 0011 = 0766
```

Note que para podermos entrar em um diretório e listarmos o conteúdo dele precisamos dar permissões de r e x

A permissão que temos acima então é r w x r w - r w -, aqui o usuário dono pode fazer qualquer coisa no diretório porém o grupo dono e outros podem listar o que tem no diretório porém não podem entrar no diretório.

Como que vai ser o calculo para arquivos

```bash
0666 - 0011 = 0655
```

A permissão que temos acima é r w - r - x r - x porém o arquivo não vai ser criada com a permissão de execução

Vamos criar um diretório para visualizarmos a nossa alteração

```bash
mkdir /srv/teste2
```

Agora vamos listar as suas permissões

```bash
ls -l /srv/
total 3
drwxr-xr-x 2 maria contabilidade 1024 Dez 22 20:30 adm
drwxr-xr-x 2 root  root          1024 Dez 22 20:46 teste
drwxrw-rw- 2 root  root          1024 Dez 22 21:00 teste2
-rw-r--r-- 1 root  root             0 Dez 22 20:48 teste2.txt
-rw-r--r-- 1 root  root             0 Dez 22 20:48 teste.txt
```

Notem que a permissão do diretório seguiu a umask corretamente, agora vamos criar um arquivo

```bash
touch /srv/teste3.txt
```

Agora vamos listar as permissões

```bash
ls -l /srv/
total 3
drwxr-xr-x 2 maria contabilidade 1024 Dez 22 20:30 adm
drwxr-xr-x 2 root  root          1024 Dez 22 20:46 teste
drwxrw-rw- 2 root  root          1024 Dez 22 21:00 teste2
-rw-r--r-- 1 root  root             0 Dez 22 20:48 teste2.txt
-rw-rw-rw- 1 root  root             0 Dez 22 21:01 teste3.txt
-rw-r--r-- 1 root  root             0 Dez 22 20:48 teste.txt
```

Note que o arquivo ficou com as mesmas permissões do diretório pois caso tenhamos permissão de escrita neste arquivo podemos mudar setar a permissão de execução mas o sistema não vai criar um arquivo executável por padrão no sistema.
