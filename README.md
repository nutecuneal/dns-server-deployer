# DNS Server Deployer

Este repositório aborda sobre como instalar uma infraestrutura de/para servidor de DNS utilizando Pi-hole e Unbound DNS.

O [**Pi-hole**](https://pi-hole.net) é um bloqueador de DNS em nível de rede privada. Atuando como um buraco negro de DNS, com ele é possível proteger os dispositivos da rede contra conteúdos indesejados e, opcionalmente, pode usá-lo como um servidor DHCP. Além disso, possui uma interface Web informativa por onde é possível gerenciar e colher estatísticas de uso da ferramenta.  [Documentação Oficial da Ferramenta¹](https://docs.pi-hole.net).

[**Unbound**](https://www.nlnetlabs.nl/projects/unbound/about/) é uma ferramente de validação, recursiva, e cache para resolução de DNS desenvolvida pela NLnet Labs. Ele é projetado para ser rápido e enxuto e incorpora recursos modernos baseados em padrões abertos. [Documentação Oficial da Ferramenta²](https://unbound.docs.nlnetlabs.nl/en/latest/). <<[Pi-hole com Docker - Github](https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker)>>.

## Sumário

- [DNS Server Deployer](#dns-server-deployer)
  - [Sumário](#sumário)
  - [Requisitos e Dependências](#requisitos-e-dependências)
  - [Instalação](#instalação)
    - [Estrutura de Diretórios](#estrutura-de-diretórios)
    - [Construção da Imagem - Docker](#construção-da-imagem---docker)
      - [Volumes](#volumes)
      - [Portas](#portas)
      - [Variáveis de Ambiente (Environment)](#variáveis-de-ambiente-environment)
      - [Rede](#rede)
      - [Executando o Docker-Compose](#executando-o-docker-compose)

## Requisitos e Dependências

- [Docker e Docker-Compose](https://docs.docker.com/)

## Instalação

### Estrutura de Diretórios

```bash
# Crie os diretórios - Pi-hole

# Dir. config.1
$ mkdir $(pwd)/etc-pihole

# Dir. config.2
$ mkdir $(pwd)/etc-dnsmasq.d

# Dir. Log
$ mkdir $(pwd)/log-pihole
```

```bash
# Crie os diretórios - Unbound

# Dir. config
$ mkdir $(pwd)/etc-unbound

# Dir. Log
$ mkdir $(pwd)/log-unbound

```

```bash
# Crie os arquivos extras de config.

# Entre na pasta
cd $(pwd)/etc-unbound

# Crie os arquivos
touch a-records.conf srv-records.conf forward-records.conf
```

Sugestão (no Linux):
- Dir. config.1 (Pi-hole): */etc/pihole*
- Dir. config.2 (Pi-hole): */etc/dnsmasq.d*
- Dir. Log (Pi-hole): */var/log/pihole*
- Dir. config (Unbound): */etc/unbound*
- Dir. Log (Unbound): */var/log/unbound*

### Construção da Imagem - Docker

#### Volumes

```yml
# docker-compose.yml (Em services.pihole-app)
# Aponte para as pastas criadas anteriormente.

# Antes
volumes:
  - '$(pwd)/etc-pihole:/etc/pihole'
  - '$(pwd)/etc-dnsmasq.d:/etc/dnsmasq.d'
  - '$(pwd)/log-pihole/:/var/log/pihole'

# Depois (exemplo)
volumes:
  - '/etc/pihole:/etc/pihole'
  - '/etc/dnsmasq.d:/etc/dnsmasq.d'
  - '/var/log/pihole:/var/log/pihole
```

```yml
# docker-compose.yml (Em services.unbounddns-app)
# Aponte para as pastas criadas anteriormente.

# Antes
volumes:
  - '$(pwd)/etc-unbound:/opt/unbound/etc/unbound'
  - '$(pwd)/log-unbound:/var/log/unbound'

# Depois (exemplo)
volumes:
  - '/etc/unbound:/opt/unbound/etc/unbound'
  - '/var/log/unbound:/var/log/unbound'
```

#### Portas

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

Obs.: A configuração abaixo só é necessária caso pretenda usar o Unbound como o serviço principal de DNS, eliminando assim o Pi-hole da arquitetura. 

```yml
# docker-compose.yml (Em services.unbounddns-app)

# Descomente (e/ou altere) as portas/serviços que você deseja oferecer. 

ports:
# Porta para DNS usando TCP.
  - '53:53'
# Porta para DNS usando UDP.
  - '53:53/udp'
```

#### Variáveis de Ambiente (Environment)

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

#### Rede

```yml
# docker-compose.yml (Em networks.dns-server-net.ipam)
# Altere o valores caso necessário. 

config:
# Endereço da rede
  - subnet: '172.18.0.0/28'
# Gateway da rede
    gateway: 172.18.0.1
```

#### Executando o Docker-Compose
```bash
# Execute

$ docker-compose -f docker-compose.yml up
```