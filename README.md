<p align="center">
<img src="https://cwmkt.com.br/wp-content/uploads/2024/04/logo_github.png" width="240" />
<p align="center">Seja bem-vindo ao Guia de Instalação Vaulwarden 🚀</p>
</p>
  
<p align="center"> 
<a href="https://hubconnect.top" target="_blank">👉 Participe da Comunidade HubConnect 👈</a>
</p>

<hr />
<hr />

### Caso não tenha Portainer e Traefix instalado, siga primeira etapa

<details>
<summary>Instalando Portainer e Traefix</summary>

### Atualizando Dependências

Atualize os repositórios do Ubuntu executando o seguinte comando:

```bash
sudo apt update && apt upgrade -y
```

----------------------------------------------------------------------------

**Instale o Docker em sua VPS**

```bash
sudo apt install docker.io -y
```

----------------------------------------------------------------------------

**Instalando Portainer**

```bash
docker swarm init
```

```bash
nano traefik.yml
```

```bash
version: "3.8"

services:

  traefik:
    image: traefik:v2.11.0
    command:
      - "--api.dashboard=true"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.network=ecosystem_network"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.web.http.redirections.entrypoint.permanent=true"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencryptresolver.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencryptresolver.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencryptresolver.acme.email=contato@seudominio.com.br"
      - "--certificatesresolvers.letsencryptresolver.acme.storage=/etc/traefik/letsencrypt/acme.json"
      - "--log.level=DEBUG"
      - "--log.format=common"
      - "--log.filePath=/var/log/traefik/traefik.log"
      - "--accesslog=true"
      - "--accesslog.filepath=/var/log/traefik/access-log"
    deploy:
      placement:
        constraints:
          - node.role == manager
      labels:
        - "traefik.enable=true"
        - "traefik.http.middlewares.redirect-https.redirectscheme.scheme=https"
        - "traefik.http.middlewares.redirect-https.redirectscheme.permanent=true"
        - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
        - "traefik.http.routers.http-catchall.entrypoints=web"
        - "traefik.http.routers.http-catchall.middlewares=redirect-https@docker"
        - "traefik.http.routers.http-catchall.priority=1"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "traefik_certificates_volume:/etc/traefik/letsencrypt"
    ports:
      - target: 80
        published: 80
        mode: host
      - target: 443
        published: 443
        mode: host
    networks:
      - ecosystem_network

volumes:
  traefik_certificates_volume:
    external: true
    name: traefik_certificates_volume

networks:
  ecosystem_network:
    external: true
    name: ecosystem_network
 ```

```bash
nano portainer.yml
```

```bash
version: "3.8"

services:

  agent:
    image: portainer/agent:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - ecosystem_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:latest
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    volumes:
      - portainer_volume:/data
    networks:
      - ecosystem_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=ecosystem_network"
        - "traefik.http.routers.portainer.rule=Host(`painel.seudominio.com.br`)"
        - "traefik.http.routers.portainer.entrypoints=websecure"
        - "traefik.http.routers.portainer.priority=1"
        - "traefik.http.routers.portainer.tls.certresolver=letsencryptresolver"
        - "traefik.http.routers.portainer.service=portainer"
        - "traefik.http.services.portainer.loadbalancer.server.port=9000"

networks:
  ecosystem_network:
    external: true
    attachable: true
    name: ecosystem_network

volumes:
  portainer_volume:
    external: true
    name: portainer_volume
```

```bash
docker network create --driver=overlay ecosystem_network
```

```bash
docker stack deploy --prune --resolve-image always -c traefik.yml traefik
```

```bash
docker stack deploy --prune --resolve-image always -c portainer.yml portainer
```

Acesse URL de seu Site e Crie Usuario
</details>


Postgres

### Adicione Stack abaixo, Stack > add stack

![image](https://github.com/cwmkt/dockerquepasa/assets/91642837/623a6dc6-c231-4105-9a02-3070d894adb8)

```bash
version: "3.8"

services:

  postgresql:
    image: bitnami/postgresql:15
    restart: always
    environment:
      POSTGRESQL_USERNAME: postgres
      POSTGRESQL_PASSWORD: r45796yv3bhub9w4f3ga3ikxmxos648r
    networks:
      - ecosystem_network
    ports:
      - 5432:5432
    volumes:
      - postgresql_volume:/bitnami/postgresql
      - postgresql_config_volume:/opt/bitnami/postgresql/conf
    deploy:
    

volumes:
  postgresql_volume:
    external: true
    name: postgresql_volume
  postgresql_config_volume:
    external: true
    name: postgresql_config_volume
    
networks:
  ecosystem_network:
    external: true
    name: ecosystem_network
```

Depois clique em DEPLOY

![image](https://github.com/cwmkt/dockerquepasa/assets/91642837/bdc62781-993a-4d31-b8cd-5cd6466900f5)

Implementação alternativa da API do servidor Bitwarden escrita em Rust e compatível com clientes Bitwarden upstream*, perfeita para implantação auto-hospedada onde executar o serviço oficial com muitos recursos pode não ser ideal.

### Passo 01: Criação do banco de dados (postgres)

1- Crie um novo database (vazio):

```bash
CREATE DATABASE vaultwarden;
```

### Passo 02: Subindo stack no Portainer

```bash
version: '3.8'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    # container_name: vaultwarden
    # restart: always

    environment:  
      DOMAIN: https://${DOMAIN_HOST}
      # SIGNUPS_ALLOWED: 'false'
      DATABASE_URL: postgresql://postgres:r45796yv3bhub9w4f3ga3ikxmxos648r@postgres:5432/vaultwarden

    networks:
      - sua_network

    volumes:
      - vaultwarden:/data

    deploy:    
      labels:
        - traefik.enable=true
        - traefik.docker.network=sua_network
        - traefik.http.routers.vaultwarden.rule=Host(`${DOMAIN_HOST}`)
        - traefik.http.routers.vaultwarden.tls=true
        - traefik.http.routers.vaultwarden.entrypoints=web,websecure
        - traefik.http.routers.vaultwarden.tls.certresolver=letsencryptresolver
        - traefik.http.routers.vaultwarden.service=vaultwarden
        - traefik.http.routers.vaultwarden.priority=1      
        - traefik.http.middlewares.vaultwarden.headers.STSSeconds=315360000
        - traefik.http.middlewares.vaultwarden.headers.browserXSSFilter=true
        - traefik.http.middlewares.vaultwarden.headers.contentTypeNosniff=true
        - traefik.http.middlewares.vaultwarden.headers.SSLHost=${DOMAIN_HOST}
        - traefik.http.middlewares.vaultwarden.headers.STSPreload=true
        - traefik.http.services.vaultwarden.loadbalancer.server.port=80

volumes:
  vaultwarden:
    external: true

networks:
  sua_network:
    external: true
```

### Ajuste Environment variables

Adicone a variavél `DOMAIN_HOST` apontando para seu dominio que será o acesso para sua instancia Vaultwarden:

```bash
DOMAIN_HOST=vaultwarden.seudominio.com.br
```

Ajuste a rede da stack para mesma que você usa o traefik, para poder gerar o certificado SSL e subir sua instância sem nenhum problema.

### Pronto tudo Funcionando ✅😎





