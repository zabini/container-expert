# Volumes

Containeres são efêmeros, todos as informações criadas por um container em execução são perdidas quando tal container morre, toda esse informação criada é expurgada.
Quando **volumes** são utilizados as informações ficam persistidas para que containeres posteriormente criados acessem tais informações. Logo, através da utilização de volumes, é possível armazenar informações de um container que execute um banco de dados por exemplo.

Volume é uma forma de "colocar um filesystem dentro de um container", porém por baixo dos panos o que ocorre é o isolamento(já que tudo no docker se trata disso) de uma parte do principal da máquina para que aquele container armazene suas informações.

## Tipos de Volumes:

### Bind

Tipo usado para montar uma pasta existente para dentro do container. (Literalmente vinculo uma pasta da minha máquina para dentro do container. Ou seja, tudo o que acontecer dentro da pasta mapeada para dentro do container, será refletido na pasta fora dele).

```bash
mkdir /opt/giropops # Cria um diretório na máquina principal
docker container run -ti --mount type=bind,src=/opt/giropops,dst=/giropops,ro debian  # executa um container usando a imagem do debian
                                                                                      # definindo algumas opções de montagem [--mount].
                                                                                      # [type] = Tipo da montagem
                                                                                      # [src]  = Pasta de Origem da montagem
                                                                                      # [dst]  = Destino dentro do contaienr (Se não existir ele cria)
                                                                                      # [ro]   = Valor opcional para READ ONLY
                                                                                      # Exemplo erro [ro] = touch: cannot touch 'testfile': Read-only file system

df -h # lista os devices de armazenamento montados no container

# Filesystem      Size  Used Avail Use% Mounted on
# overlay         212G  143G   58G  72% /
# tmpfs            64M     0   64M   0% /dev
# tmpfs           3.8G     0  3.8G   0% /sys/fs/cgroup
# shm              64M     0   64M   0% /dev/shm
# /dev/sdb1       212G  143G   58G  72% /giropops           [Repare no volume montado como um device]
# tmpfs           3.8G     0  3.8G   0% /proc/asound
# tmpfs           3.8G     0  3.8G   0% /proc/acpi
# tmpfs           3.8G     0  3.8G   0% /sys/firmware
```

### Volume

```bash
docker volume ls # lista os volumes existentes
docker volume create [VOLUME NAME]  # cria um volume
docker volume inspect [VOLUME NAME] # inspeciona o volume definido

# [
#     {
#         "CreatedAt": "2021-03-31T01:23:10-03:00",
#         "Driver": "local",
#         "Labels": {},
#         "Mountpoint": "/var/lib/docker/volumes/giropops/_data", # real path onde os dados ficam armazenados
#         "Name": "giropops",
#         "Options": {},
#         "Scope": "local"
#     }
# ]

docker container run -ti --mount type=volume,src=giropops,dst=/giropops debian 
                                      # Aloca um volume criado previamente para um container 
                                      # Isso permite que haja o compartilhamento de volumes entre containeres

docker volume rm [VOLUME NAME]  # Remove o volume. Porém caso tenha algum container 
                                # parado que utiliza este volume a remoção do volume falhará.
                                # A remoção de uma container expurga completamenta as informações
                                # Isso é apaga os dados da pasta: [/var/lib/docker/volumes]
```

## Prune

```bash
docker volume prune # Remove os volumes que não são utilizados por pelo menos um container
```

## Data-only container

Compartilhamento de volumes entre containeres, parametro = [--volumes-from]

```bash
docker container create -v /data --name dbdados centos # Cria um container 

docker container run -d \
  -p 5432:5432 --name pgsql1 --volumes-from dbdados \
  -e POSTGRESQL_USER=docker \
  -e POSTGRESQL_PASS=docker \
  -e POSTGRESQL_DB=docker kamui/postgresql

docker container run -d \
  -p 5433:5432 --name pgsql2 --volumes-from dbdados \
  -e POSTGRESQL_USER=docker \
  -e POSTGRESQL_PASS=docker \
  -e POSTGRESQL_DB=docker kamui/postgresql
```

# Desafio

```bash
docker volume create dbdados # Criação do volume nomeado como dbdados
docker volume inspect dbdados
# [
#     {
#         "CreatedAt": "2021-04-01T01:58:10-03:00",
#         "Driver": "local",
#         "Labels": {},
#         "Mountpoint": "/var/lib/docker/volumes/dbdados/_data",
#         "Name": "dbdados",
#         "Options": {},
#         "Scope": "local"
#     }
# ]

docker container run -d \
  --mount type=volume,src=dbdados,dst=/data \
  -p 5432:5432 --name pgsql1 \
  -e POSTGRESQL_USER=docker \
  -e POSTGRESQL_PASS=docker \
  -e POSTGRESQL_DB=docker kamui/postgresql

docker container run -d \
  --mount type=volume,src=dbdados,dst=/data \
  -p 5433:5432 --name pgsql2 \
  -e POSTGRESQL_USER=docker \
  -e POSTGRESQL_PASS=docker \
  -e POSTGRESQL_DB=docker kamui/postgresql
```

# Backup

```bash

mkdir /opt/backup # Cria um diretório para armazenar os backups

docker container run -ti \
  --mount type=volume,src=dbdados,dst=/data \
  --mount type=bind,src=/opt/backup,dst=/backup \
  debian tar -cvf /backup/bkp-banco.tar /data # Cria um container para executar o backup de um volume
```