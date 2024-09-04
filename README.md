
# WordPress Multi-Instance com Replicação MySQL

Este projeto configura um ambiente Docker que hospeda várias instâncias do WordPress, cada uma com seu próprio banco de dados MySQL isolado e replicado para um MySQL principal. Isso garante alta disponibilidade, redundância dos dados e isolamento total entre os sites.

## Explicação Detalhada

### Isolamento de Redes

- **wordpress1_network:** Cada instância do WordPress tem sua própria rede interna, comunicando-se exclusivamente com seu MySQL dedicado.
- **traefik_bridge:** Conecta o WordPress e o MySQL interno ao Traefik e à replicação com o MySQL principal.

### MySQL Interno com Replicação

- **wordpress1_mysql:** Banco de dados dedicado para a instância do WordPress, isolado na **wordpress1_network**.
- **mysql_replica:** Serviço configurado como escravo (slave) que replica os dados do MySQL interno. A replicação ocorre através da **traefik_bridge**, onde o MySQL interno é o master.

### Balanceamento de Carga

Você pode usar o Traefik para rotear conexões de banco de dados ou configurar o WordPress para se conectar a uma lista de hosts MySQL, mas o ideal é centralizar isso no Traefik.

### Volumes

- **wordpress1_data:** Armazena arquivos do WordPress (temas, plugins, uploads). Se quiser mais velocidade, use um bucket com MinIO na sua VPS.
- **wordpress1_mysql_data:** Armazena os dados do banco de dados MySQL local.

### Replicação MySQL

A replicação permite duplicar os dados do banco em outra instância, garantindo redundância. Isso é feito pelo container **mysql_replica**, que replica os dados do **wordpress1_mysql**.

### Automação e Failover

Para uma replicação automática e failover entre o MySQL local e o principal, você pode:

- **Monitorar e Automatizar:** Use ferramentas como Orchestrator ou MHA para gerenciar failovers e promover slaves a masters automaticamente.
- **Balanceamento Dinâmico:** Ferramentas como ProxySQL podem balancear a carga e rotear conexões entre múltiplos MySQL.

### Expansão para Novas Instâncias

Para adicionar novas instâncias do WordPress, basta replicar os blocos de código de **wordpress1**, **wordpress1_mysql**, e **mysql_replica**, alterando os nomes e portas conforme necessário.

> **IMPORTANTE:** Este projeto é ideal para desenvolvimento e testes. Se for usar em produção, entenda bem as implicações de segurança e desempenho.

## Requisitos de Software Mínimo

- **Sistema Operacional:** Testado em Ubuntu 22.04
- **Docker:** 20.x ou superior
- **Docker Compose:** 1.27.x ou superior
- **Acesso à Internet:** Para baixar imagens Docker e configurar SSL
- **Hardware Recomendado:** (ver abaixo)

## Requisitos Recomendados de Hardware

### CPU

- **Mínimo:** 4 núcleos
- **Recomendado:** 8 núcleos ou mais

### Memória RAM

- **Mínimo:** 8 GB
- **Recomendado:** 16 GB ou mais

### Armazenamento

- **Mínimo:** 100 GB SSD
- **Recomendado:** 250 GB SSD ou mais

### Rede

- **Recomendado:** Conexão de alta velocidade (1 Gbps ou mais)

### Backup e Redundância

- **Recomendado:** Backup regular e redundância de armazenamento (use MinIO)

## Passos para Configurar o Ambiente

### 1. Instalar Docker e Docker Compose

No Ubuntu 22.04, use os seguintes comandos:

```bash
sudo apt-get update
sudo apt-get install docker.io docker-compose -y
```

### 2. Clonar o Repositório

```bash
git clone https://github.com/Pixel-Guru/WordPress-Multi-Instance-with-MySQL-Replication
```

### 3. Configurar Redes e Volumes

```bash
docker network create --driver=bridge principal
docker network create --driver=bridge wordpress1_network
docker network create --driver=bridge traefik_bridge
```

> **Nota:** Os volumes serão gerados automaticamente pelo Docker durante o deploy.

### 4. Configurar o Arquivo .env

Crie um arquivo .env na raiz do projeto:

```bash
touch .env
```

### 4.1 Preencha o arquivo .env

```bash
WORDPRESS_DB_PASSWORD=wordpress1_pass
MYSQL_ROOT_PASSWORD=wordpress1_root_pass
MYSQL_REPLICATION_USER=replica_user
MYSQL_REPLICATION_PASSWORD=replica_pass
```

### 5. Fazer o Deploy da Stack

```bash
docker stack deploy -c docker-compose.yml wordpress_cluster
```

### 6. Monitorar os Containers

Use os comandos abaixo para monitorar os containers:

```bash
docker service ls
docker ps
```

### 7. Acessar o WordPress

Acesse o WordPress pelo domínio configurado:

```bash
http://nexizo.cloud/
```

## Estrutura do Projeto

### Serviços Principais

- **Traefik:** Proxy reverso que gerencia o roteamento HTTP/HTTPS.
- **WordPress:** Instâncias do WordPress em redes internas isoladas.
- **MySQL Interno:** Banco de dados MySQL isolado para cada instância de WordPress.
- **MySQL Replicado:** MySQL principal que recebe dados replicados.

### Redes

- **traefik_bridge:** Rede para comunicação entre o Traefik e os serviços.
- **wordpress1_network:** Rede interna isolada para cada instância do WordPress.
- **principal:** Rede principal usada para replicação de dados.

## Licença

Este projeto está sob a licença MIT. Isso significa que você pode usar, modificar e distribuir o software, desde que mantenha os créditos.

```
MIT License

Copyright (c) 2024 Alexandre Louro
```

## Autor

Projeto criado por Alexandre Louro. Dúvidas ou colaborações? Entre em contato em contato-louro@proton.me .Se usar ou modificar este projeto, mantenha os créditos.

## Contribuições

Contribuições são bem-vindas! Abra um pull request ou issue no GitHub para discutir mudanças ou melhorias.

**Gostou do projeto?** Faça uma doação pelo Pix: **82 9 9838-7735**.
