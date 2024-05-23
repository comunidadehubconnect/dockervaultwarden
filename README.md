<p align="center">
<img src="https://cwmkt.com.br/wp-content/uploads/2024/04/logo_github.png" width="240" />
<p align="center">Seja bem-vindo ao Guia de Instalação Vaulwarden 🚀</p>
</p>
  
<p align="center"> 
<a href="https://hubconnect.top" target="_blank">👉 Participe da Comunidade HubConnect 👈</a>
</p>

<hr />
<hr />

Implementação alternativa da API do servidor Bitwarden escrita em Rust e compatível com clientes Bitwarden upstream*, perfeita para implantação auto-hospedada onde executar o serviço oficial com muitos recursos pode não ser ideal.

### Passo 01: Criação do banco de dados (postgres)

1- Crie um novo database (vazio):

```bash
CREATE DATABASE vaultwarden;
```

2- Crie um usuário e garanta os previlégios:

```bash
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD '7223y2i9ruxqpaaai539ww454n8gi3xq';
GRANT ALL ON DATABASE vaultwarden TO vaultwarden;
GRANT all privileges ON database vaultwarden TO vaultwarden;
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





