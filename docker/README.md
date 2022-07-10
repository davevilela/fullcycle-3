# Resumão Full cycle docker

1. [Conceitos docker](#conceitos)
   1. [Host](#host)
   2. [Client](#client)
   3. [Containers](#containers)
   4. [Processos](#processos)
   5. [Networks](#networks)
   6. [Volumes](#volumes)
   7. [Imagens](#imagens)
   8. [Dockerfile](#dockerfile)

# Conceitos docker <a name="conceitos"></a>

## **o Docker é composto por:**

- Namespaces
- CGroups
- File system (OFS)

## Docker host -> daemon em background <a name="host"></a>

- Cache
- O docker host cacheia as builds numa estrutura chamada registry
- Volumes: Persistencia de dados entre o host e o client
- Containers são imutáveis
- Cada container é um processo
- Network: Comunicação entre processos (containers)

## Docker client -> App do docker no terminal <a name="client"></a>

- Run
- Pull
- Push

```bash
# rodar o docker em modo interativo
docker run -i ...

# rodar o docker com tty (conseguir rodar comandos)
docker run -t ...
```

## Containers <a name="containers"></a>

```bash
# iniciar um container
docker start [nome do processo]

# remover o container depois de finalizar o processo
docker run -rm [nome da imagem] [nome do comando]

# rodar um container em modo "detached", que faz o terminal ficar livre
docker run -d [container ou imagem] ...

#parar o processo de um container
docker stop [id do processo]

#remover um container
docker rm [-f] [id ou nome do processo]

#executar comandos dentro de um container
docker exec [id ou nome do processo] [...]
docker exec -it nginx bash

```

## Processos <a name="processos"></a>

```bash
# ver todos os processos ativos no docker host
docker ps

# ver todos os processos, inclusive os que já morreram
docker ps -a

# ver somente ids de todos os containers
docker ps -a -q

# apagar todos os containers
docker rm $(docker ps -a -q) -f

# attach em um processo
docker attach [nome do container]
# outra opão é usar o seguinte comando:
docker exec [nome do container]
# exemplo 
docker exec -it ubuntu1 bash
```

## Networks <a name="networks"></a>

comandos:
```bash
# redirecionamento de porta entre o container e o docker-host
# a primeira porta é do host, a segunda do container
docker run -p 8080:80 nginx

# comando base
docker network

# listar todas as networks
docker network ls

# remover networks que não estão sendo utilizadas
docker network prune

#inspecionar rede
docker inspect [nome da rede]

# criar nova rede
docker network create --driver [tipo do driver] [nome da rede]

# exemplo
docker network create --driver bridge minharede

# criar um container com uma network customizada
docker run [...] --network [nome da rede] [nome da imagem]
# exemplo
docker run -dit --name ubuntu1 --network minharede bash

# conectando um container a uma rede específica
docker network connect [nome da rede] [nome do container]
# exemplo
docker network connect minharede ubuntu3
```


tipos de network (drivers):
- **bridge**: Usamos para um container se comunicar com outro.
- **host**: permite que o container acesse uma porta da máquina host (no caso, a máquina que está rodando o docker host.) Geralmente é usado no lugar do bind mount de portas. 
- **overlay**: usado para comunicar vários hosts (máquinas). Usado pelo docker swarm.
- **maclan**: mapeia o container para uma porta física através de um endereco mac. Não é comumente utilizado.
- **none**: o container fica isolado, sem fazer parte de nenhuma rede.

Tipos mais utilizados:
- Brige
- Host
- Overlay (quando usando docker swarm)
 
 Obs: por padrão, todos os containers são iniciados com uma network "bridge" do docker.
## Volumes <a name="volumes" /></a>

```bash
# mapear um volume local para o armazenamento do container (bind mount)
# primeiro path é o volume local
# segundo path é o diretorio do container
# com comando antigo (-v)
docker run [...] -v ~/projects/fullcycle/docker/html/:/usr/share/nginx/html
#ou
docker run [...] -v "$(pwd)"/html:/usr/share/nginx/html
# com comando --mount
# type=bind para informar que você quer associar um diretório
docker run [preferencias] --mount type=bind,source="${pwd}"/html,target=[volume destino] [imagem]
```

## Imagens <a name="imagens"></a>

```bash
# baixar imagem do container registry (docker hub)
# (você pode ter um container registry próprio e privado)
docker pull [nome da imagem]

# listar imagens presentes localmente
docker images

# remover imagem
docker image rm [nome da imagem]
# ou
docker rmi [nome da imagem]
```

obs: sempre criamos imagens a partir de imagens existentes. :joy:

### Builds de imagem

a partir de um diretório onde você tenha um arquivo **Dockerfile**
digite o comando:

```bash
> docker build -t davivilela/nginx-com-vim:latest .
```

**obs**: 

**"davivilela"** é o usuário do docker hub

**"nginx-com-vim:latest"** é o nome da minha imagem

**"."** é o diretório onde está a imagem.

## Dockerfile <a name="dockerfile"></a>

**criar um diretório dentro do container quando for iniciado:**
```Dockerfile
WORKDIR /app
```

**copiar aqruivo do diretório local para um diretório dentro do container**

```Dockerfile
COPY html /usr/share/nginx/html
```
onde o primeiro parâmetro é o diretório local, e o segundo é o diretório do container.

**selecionar usuário**
```Dockerfile
USER [nome do usuário]
```

### Entrypoint vs CMD

**usando o comando CMD**
```Dockerfile
FROM ubuntu:latest
CMD ["echo", "hello world"]
```

quando usamos o **CMD** qualquer parâmetro passado no "docker run" substituirá
os comandos estabelecidos no Dockerfile.

**usando o comando ENTRYPOINT**
```Dockerfile
ENTRYPOINT ["echo", "Hello "]

CMD ["World"]
```
o **ENTRYPOINT** é um comando fixo, diferente do CMD que pode ser usado como parâmetro do entrypoint.


### Expondo uma porta

exemplo:
```Dockerfile
EXPOSE 80
```

## Publicando uma imagem no Docker hub

```bash
docker push [nome de usuario]/[nome da imagem]
# exemplo:
docker push davivilela/nginx:latest
```