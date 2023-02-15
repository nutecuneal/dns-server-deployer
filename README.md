# DNS Server Deployer

Este repositório aborda sobre como instalar uma infraestrutura de/para servidor de DNS utilizando Pi-hole e Unbound DNS.

O [**Pi-hole**](https://pi-hole.net) é um bloqueador de DNS em nível de rede privada. Atuando como um buraco negro de DNS, com ele é possível proteger os dispositivos da rede contra conteúdos indesejados e, opcionalmente, pode usá-lo como um servidor DHCP. Além disso, possui uma interface Web informativa por onde é possível gerenciar e colher estatísticas de uso da ferramenta.  [Documentação Oficial da Ferramenta¹](https://docs.pi-hole.net).

[**Unbound**](https://www.nlnetlabs.nl/projects/unbound/about/) é uma ferramente de validação, recursiva, e cache para resolução de DNS desenvolvida pela NLnet Labs. Ele é projetado para ser rápido e enxuto e incorpora recursos modernos baseados em padrões abertos. [Documentação Oficial da Ferramenta²](https://unbound.docs.nlnetlabs.nl/en/latest/). <<[Pi-hole com Docker - Github](https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker)>>.

## Sumário

- [DNS Server Deployer](#dns-server-deployer)
  - [Sumário](#sumário)
  - [Requisitos e Dependências](#requisitos-e-dependências)
  - [Instalação](#instalação)
    - [PiHole](#pihole)
      - [Estrutura de Diretórios](#estrutura-de-diretórios)
      - [Docker-Compose](#docker-compose)
        - [Portas](#portas)
        - [Volumes](#volumes)
        - [Variáveis de Ambiente (Environment)](#variáveis-de-ambiente-environment)
    - [Unbound](#unbound)
      - [Estrutura de Diretórios](#estrutura-de-diretórios-1)
      - [Docker-Compose](#docker-compose-1)
        - [Portas](#portas-1)
        - [Volumes](#volumes-1)
    - [Docker-Compose - Rede](#docker-compose---rede)
    - [Executando o Docker-Compose](#executando-o-docker-compose)

## Requisitos e Dependências

- [Docker e Docker-Compose](https://docs.docker.com/)

## Instalação

### PiHole

#### Estrutura de Diretórios

```bash
# Crie os diretórios

# Dir. config.1
$ mkdir $(pwd)/etc-pihole

# Dir. config.2
$ mkdir $(pwd)/etc-dnsmasq.d

# Dir. Log
$ mkdir $(pwd)/log-pihole
```

Sugestão (no Linux):
- Dir. config.1: */etc/pihole*
- Dir. config.2: */etc/dnsmasq.d*
- Dir. Log: */var/log/pihole*

#### Docker-Compose

##### Portas

```yml
# docker-compose.yml (Em services.pihole-app)

# Descomente (e/ou altere) as portas/serviços que você deseja oferecer.

ports:
# Porta para DNS usando TCP.
  - '53:53'
# Porta para DNS usando UDP.
  - '53:53/udp'
# Porta de serviço DHCP.
  - '67:67/udp'
# Porta para a interface Web de administração.
  - '80:80'
```

##### Volumes

```yml
# docker-compose.yml (Em services.pihole-app)
# Aponte para as pastas criadas anteriormente.

# Antes
volumes:
  - '$(pwd)/etc_pihole:/etc/pihole'
  - '$(pwd)/etc_dnsmasq.d:/etc/dnsmasq.d'
  - '$(pwd)/log_pihole/:/var/log/pihole'

# Depois (exemplo)
volumes:
  - '/etc/pihole:/etc/pihole'
  - '/etc/dnsmasq.d:/etc/dnsmasq.d'
  - '/var/log/pihole:/var/log/pihole
```

##### Variáveis de Ambiente (Environment)

```yml
# docker-compose.yml (Em services.pihole-app)
# Altere o valores caso necessário. 

environment:
# Timezone.
  - TZ=America/Maceio
# Tamanho do cache
  - CUSTOM_CACHE_SIZE=10000 
# Servidor de DNS que o Pi-hole irá se conectar.
  - PIHOLE_DNS_=dns-server-unbound
# Senha do usuário admin.
  - WEBPASSWORD=webuserpass
```

### Unbound

#### Estrutura de Diretórios

```bash
# Crie os diretórios - Unbound

# Dir. config
$ mkdir $(pwd)/etc-unbound

# Dir. Log
$ mkdir $(pwd)/log-unbound

```

Sugestão (no Linux):
- Dir. config: */etc/unbound*
- Dir. Log: */var/log/unbound*


```bash
# Crie os arquivos extras de config (em $(pwd)/etc-unbound).

# Crie os arquivos
touch a-records.conf srv-records.conf forward-records.conf
```

#### Docker-Compose

##### Portas

Essa configuração só é necessária caso pretenda usar o Unbound como o serviço principal de DNS, eliminando assim o Pi-hole da arquitetura. 

```yml
# docker-compose.yml (Em services.unbounddns-app)

# Descomente (e/ou altere) as portas/serviços que você deseja oferecer. 

ports:
# Porta para DNS usando TCP.
  - '53:53'
# Porta para DNS usando UDP.
  - '53:53/udp'
```

##### Volumes

```yml
# docker-compose.yml (Em services.unbounddns-app)
# Aponte para as pastas criadas anteriormente.

# Antes
volumes:
  - '$(pwd)/etc_unbound:/opt/unbound/etc/unbound'
  - '$(pwd)/log_unbound:/var/log/unbound'

# Depois (exemplo)
volumes:
  - '/etc/unbound:/opt/unbound/etc/unbound'
  - '/var/log/unbound:/var/log/unbound'
```

### Docker-Compose - Rede

```yml
# docker-compose.yml (Em networks.dns-server-net.ipam)
# Altere o valores caso necessário. 

config:
# Endereço da rede
  - subnet: '172.18.0.0/28'
```

### Executando o Docker-Compose
```bash
# Execute

$ docker-compose up
```
