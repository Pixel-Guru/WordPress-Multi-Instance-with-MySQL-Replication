
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

# /

# Possíveis Razões para a Stack não Funcionar Corretamente
*/
# Configurações de Rede

Rede não Existente: Certifique-se de que todas as redes definidas no arquivo muti_wordpress_em_docker.yaml existem previamente. Use o comando docker network 
```create <network_name>``` para criar redes necessárias.

# Conflito de Nomes: 
Nomes de redes duplicados podem causar conflitos. Ao adicionar novas instâncias, sempre utilize nomes únicos para as redes.
*/
# Variáveis de Ambiente Não Definidas

# Arquivo .env Ausente ou Incompleto:
Verifique se todas as variáveis de ambiente utilizadas (WORDPRESS_DB_PASSWORD, MYSQL_ROOT_PASSWORD, MYSQL_REPLICATION_USER, MYSQL_REPLICATION_PASSWORD) estão devidamente definidas em um arquivo .env.

# Valores Inválidos ou Vazios: 
Assegure-se de que as variáveis possuem valores válidos e seguros. Valores vazios podem levar a falhas na inicialização dos serviços.
Conflitos de Portas

# Portas Já em Uso: 
Verifique se as portas utilizadas pelos serviços (como a 3306 para MySQL) não estão sendo usadas por outros processos no host. Caso necessário, mapeie para portas alternativas.
*/
# Persistência de Dados

Volumes Não Criados: Certifique-se de que todos os volumes externos especificados estão criados antes de iniciar a stack. Use

```
docker volume create <volume_name>
```
para criar os volumes necessários.
*/
# Permissões de Arquivos: 
Verifique se o usuário que executa o Docker tem permissões adequadas nos diretórios de volumes para leitura e escrita.
*/
# Configurações de DNS e Domínios

# Domínio Não Apontando para o Servidor: 
Assegure-se de que os domínios ou subdomínios especificados estão corretamente configurados e apontam para o endereço IP do servidor onde a stack está sendo executada.

# Certificados SSL: 
Problemas na obtenção ou renovação de certificados Let's Encrypt podem impedir o acesso seguro. Verifique os logs do Traefik para identificar e resolver esses problemas.
*/
# Incompatibilidades de Versões

# Versões de Imagens Desatualizadas ou Incompatíveis: 
Certifique-se de que as versões das imagens Docker utilizadas são compatíveis entre si. Por exemplo, certas versões do WordPress podem não ser totalmente compatíveis com versões específicas do MySQL.

# Atualizações Necessárias: 
Mantenha as imagens Docker atualizadas para aproveitar correções de bugs e melhorias de segurança.
Recursos Insuficientes

# Limitações de Hardware: 
Recursos insuficientes de CPU e RAM podem levar a falhas na inicialização ou desempenho degradado dos serviços. Veja a seção de requisitos de hardware abaixo para mais detalhes.
*/
# Erros de Configuração no Traefik

# Configurações de Roteamento Incorretas: 
Verifique se as labels configuradas para o Traefik estão corretas e correspondem aos serviços e domínios apropriados.

# Falta de Permissões: 
Certifique-se de que o Traefik tem acesso adequado ao Docker socket e aos arquivos de configuração necessários.
*/
# Erros e Bugs Esperados
# Conexão Recusada ao Banco de Dados

Causa: Credenciais incorretas, serviço MySQL não iniciado ou problemas de rede.
Solução: Verifique as credenciais fornecidas, certifique-se de que o serviço MySQL está em execução e que as redes estão corretamente configuradas.

# Tempo de Espera Excedido ao Acessar o Site

Causa: Traefik não roteando corretamente as solicitações, serviço WordPress não iniciado ou problemas de DNS.
Solução: Verifique as configurações do Traefik, assegure-se de que o serviço WordPress está ativo e confirme que os registros DNS estão corretos.

#Erros de Permissão nos Volumes

Causa: O Docker não tem permissões adequadas para ler ou escrever nos diretórios de volumes.
Solução: Ajuste as permissões dos diretórios correspondentes e reinicie os serviços afetados.

# Falhas na Replicação do MySQL

Causa: Configurações incorretas de replicação, usuário de replicação não configurado corretamente ou diferenças de versão entre master e slave.
Solução: Revise as configurações de replicação, confirme que o usuário e senha de replicação estão corretos e assegure-se de que as versões do MySQL são compatíveis.

# Certificados SSL Não Gerados

Causa: Limites de taxa do Let's Encrypt atingidos, configurações incorretas do certresolver ou problemas de conectividade.
Solução: Verifique os logs do Traefik para detalhes, ajuste as configurações conforme necessário e considere usar certificados autoassinados temporariamente.

#Requisitos de Hardware !importat
Os requisitos de hardware para executar esta stack de forma eficiente dependem da carga esperada e do número de instâncias que você planeja executar simultaneamente.

# Memória RAM
Importância: A RAM é crítica para o desempenho, especialmente devido à execução simultânea de múltiplas instâncias do MySQL e WordPress. Cada instância consome uma quantidade significativa de memória para operar de forma eficiente.

Recomendação:
Ambientes de Desenvolvimento/Teste: Mínimo de 4GB de RAM.

Ambientes de Produção com Baixo Tráfego: Mínimo de 8GB de RAM.

Ambientes de Produção com Alto Tráfego ou Múltiplas Instâncias: 16GB de RAM ou mais, dependendo do número de instâncias e volume de acessos.

# Processador (CPU)
Importância: A CPU é responsável por lidar com as requisições de usuários, processamento de dados e tarefas de replicação do banco de dados. Um processador mais potente garante respostas mais rápidas e melhor capacidade de processamento paralelo.

Recomendação:
Ambientes de Desenvolvimento/Teste: Processador dual-core com frequência mínima de 2.0 GHz.
Ambientes de Produção com Baixo Tráfego: Processador quad-core com frequência mínima de 2.5 GHz.

Ambientes de Produção com Alto Tráfego ou Múltiplas Instâncias: Processador hexa-core ou superior, preferencialmente com capacidades de multithreading.

# Armazenamento
Importância: Espaço de armazenamento adequado é necessário para hospedar os dados do WordPress, bancos de dados MySQL e logs do sistema.

Recomendação:
Ambientes de Desenvolvimento/Teste: Mínimo de 50GB de armazenamento SSD.

Ambientes de Produção: Mínimo de 100GB de armazenamento SSD, com possibilidade de expansão conforme o crescimento dos dados.

# Rede
Importância: Uma conexão de rede rápida e estável é essencial para garantir tempos de resposta baixos e alta disponibilidade.
Recomendação:
Largura de Banda: Mínimo de 100Mbps para ambientes de produção, preferencialmente 1Gbps para alto tráfego.
Latência: Mantenha a latência o mais baixa possível, especialmente entre os servidores que hospedam as diferentes instâncias e serviços.

# Considerações Adicionais
Escalabilidade: Considere configurar um ambiente que possa ser facilmente escalado verticalmente (adicionando mais recursos ao servidor existente) ou horizontalmente (adicionando mais servidores ao cluster) conforme a demanda aumenta.

Monitoramento: Implemente ferramentas de monitoramento para acompanhar o uso de recursos e desempenho, permitindo ajustes proativos antes que problemas ocorram.
Backup e Redundância: Estabeleça rotinas de backup regulares e considere soluções de redundância para garantir a continuidade do serviço em caso de falhas de hardware ou software.

# /

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

Projeto criado por Alexandre Louro. Dúvidas, bugs ou colaborações? Entre manda um email para: contato-louro@proton.me com o assunto especifico ex: detctei um bug houve isso e aquilo e etc.... Se usar ou modificar este projeto, mantenha os créditos.

## Contribuições

Contribuições são bem-vindas! Abra um pull request ou issue no GitHub para discutir mudanças ou melhorias.

**Gostou do projeto?** Faça uma doação pelo Pix: **82 9 9838-7735**.
