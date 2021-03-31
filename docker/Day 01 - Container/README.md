# Container

Container é uma forma de fazer isolamento! Isolar processos, e outros recursos.
Ele utiliza o kernel do próprio host, para o gerenciamento dos recursos.

## Tipos de Isolamento 
  - Isolamento lógico:
    - Rede, Usuários e Processos

  - Isolamento Físico
    - Recursos de Hardware como CPU, memória, IO de Rede

## Namespaces
  
  - PID namespaces: Isolamento de Processos
  - Net Namespaces: Isolamento das Redes, interfaces de rede.
  - MNT Namespaces: Gerencia os mount points
  - User Namespaces: Isolamento de usuários

**CGROUPS** Responsável por fazer o isolamento de recursos como CPU, network, disco, memória e etc.

## Imagens

As imagens no docker são baseada em camadas. As quais somente a última camada é a de escrita(**Copy on Write**). Ou seja, todo arquivo que estão nas camadas inferiores caso seja solicitado uma alteração neste, a ação é realizada em uma cópia e não no arquivo original.

Isso ocorre para garantir a integridade das informações de uma imagem. Se houve a necessidade de replicar o container, com o Copy on Write há a garantia que todos os containeres estão rodando com a mesma informação. Ou seja, isso permite o compartilhamento de imagens compartilhadas por containeres em execução ex: 2 containeres usando a mesma imagem. O que muda é a última camada e os dados que um container gera

Geralmente na ultima camada os arquivos alterados são menos importantes(na teoria), como arquivos de logs por exemplo.

## Prática

### Install:
```bash
curl -fsSL https://get.docker.com | bash
sudo usermod -aG docker your-user # Usuário que poderá gerenciar o docker
```

### Versão:
```bash
docker version
```

### Listar containers
Parâmetro **`-a`** inclui os cone containeres mortos na listagem
```bash
docker container ls -a
```

### Running Hello World
Parâmetros
- **`-t`** Terminal
- **`-i`** Interatividade, quando o container for executado. Automaticamente já estara atached ao terminal.
```bash
docker container run -ti hello-world
```

### Running Ubuntu
```bash
docker container run -ti ubuntu
```

### Running Centos
```bash
docker container run -ti centos
```

### Entrypoint

Todo container tem um **Entrypoint**. Ele é o principal processo do container, caso este processo por algum motivo ele termine, o docker entende que tal container não tem mais necessidade de estar vivo. A partir disso o container é terminado.
No caso das imagens executadas acima, os principais processos é a execução do terminal(`bash`).
Para sair do modo interativo sem causar uma interrupção no container. Deve-se executar a seguinte combinação de teclas: **`ctrl + P + Q`**

### Attach to container
Conecta-se ao container em execução
```bash
docker container attach <id-do-container>
```

### Nginx container

Executa a imagem do nginx

```bash
docker container run -ti nginx
```
Neste caso ele se conectará ao processo no nginx. Os processos dentro do docker precisam estar em execução em primeiro plano(foreground), ou seja, não podem ser executados como daemon. Eles precisam ser iniciados via o **entrypoint** da imagem.


Executa o container como um daemon

```bash
docker container run -d nginx
```

Executa um comando dentro do container no modo interativo. Como o comando `bash` invoca o terminal do container. Você pode executar interativamente vários comandos dentro de um container.

### Exec

```bash
docker container exec -ti [CONTAINER ID] [COMANDO]

# Exemplo
docker container exec -ti 57635202c7bd bash
curl localhost # examplo, chama a index do nginx
```

### Base commands

```bash
docker container start [CONTAINER ID]    # Inicia o container
docker container stop [CONTAINER ID]     # Para o container
docker container restart [CONTAINER ID]  # Rinicia o container
docker container pause [CONTAINER ID]    # Pausa a execução do container
docker container unpause [CONTAINER ID]  # Despausa a execução do container
docker container rm [CONTAINER ID] -f    # remove um container. -f para forçar a remoção(caso em execução)
docker container inspect [CONTAINER ID]  # Inspeciona os detalhes de container
docker container logs -f [CONTAINER ID]  # Inspeciona os detalhes de container
```

### Inspeção e configuração de recursos

```bash
docker container stats [CONTAINER ID] # Status de uso dos recursos. 
                                      # Ex: Usando imagem nginx [while true; do curl 172.17.0.2; done;]
                                      # Note o uso do I/O de rede
                                      # Ex: [apt-get update && apt-get install stress]
                                      # [stress --cpu 1 --vm-bytes 128M --vm 1]

docker container top [CONTAINER ID]   # Exibe os proocessos em execução na máquina

docker container run -d -m 128M nginx # Executa um container usando a imagem do nginx. inspect: [ "Memory": 134217728, ]
                                      # -m para limitar a quantidade de memória utilizad pelo container
                                      # [stress --vm 1 --vm-bytes 150M]
                                      # O container não excederá 128bm de uso de memória

docker container run -d -m 128M --cpus 0.5 nginx  # --cpus indica a quantidade de cores usadas na 
                                                  # execução do container. inspect: [ "NanoCpus": 500000000, ]
                                                  # Ver quantidade de cors de CPU: [ cat /proc/cpuinfo ]
                                                  # [stress --cpu 1 --vm 1 --vm-bytes 64M]
                                                  # O resultado esperado é ocupar somente 1 core de CPU da máquina

docker container update --cpus 0.2 --memory 64M [CONTAINER ID]  # atualiza a quantidade de recursos que um 
                                                                # container em EXECUÇÃO pode utilizar 
```

### Imagens

```bash
docker image build -t toskeira:1.0
docker image ls
docker container run -d toskeira:1.0
docker container logs -f [CONTAINER ID]

# No Dockerfile:
# 
# FROM debian 
# LABEL app="Giropops" 
# ENV JEFERSON="LINDO" 
# RUN apt-get update && apt-get install -y stress && apt-get clean 
# 
# CMD stress --cpu 1 --vm-bytes 64M --vm1
```