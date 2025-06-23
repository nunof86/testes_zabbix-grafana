✅ Instalação do Zabbix
Instalar Git (caso ainda não estivesse instalado):

```bash
sudo apt install git
```

Clonar o repositório oficial:

```bash
git clone https://github.com/zabbix/zabbix-docker
cd zabbix-docker
```

Subir o ambiente Zabbix com Docker Compose:

```bash
docker compose -f ./docker-compose_v3_ubuntu_mysql_local.yaml up -d
```

✅ Instalação e automação do Grafana

Estrutura de diretórios e ficheiros:
```bash
grafana/
├── docker-compose.yml
├── enable_zabbix_plugin.sh
└── provisioning
    └── datasources
        └── zabbix.yaml
```

**docker-compose.yml**
```yaml
services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./provisioning:/etc/grafana/provisioning
      - ./enable_zabbix_plugin.sh:/etc/grafana/enable_zabbix_plugin.sh
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_INSTALL_PLUGINS=alexanderzobnin-zabbix-app
    entrypoint: ["/bin/bash", "-c", "/run.sh & \
      until curl -s http://localhost:3000/api/health | grep '\"database\": \"ok\"' > /dev/null; do \
        echo 'Aguardando Grafana iniciar completamente...'; \
        sleep 5; \
      done; \
      echo 'Grafana pronto. Ativando plugin...'; \
      /etc/grafana/enable_zabbix_plugin.sh; \
      wait"]
    restart: unless-stopped

volumes:
  grafana_data:
```

**enable_zabbix_plugin.sh**
```bash
#!/bin/bash

curl -X POST http://localhost:3000/api/plugins/alexanderzobnin-zabbix-app/settings \
  -H "Content-Type: application/json" \
  -u admin:admin \
  -d '{ "enabled": true }'
```
✅ Este script é chamado automaticamente após o arranque do Grafana para ativar o plugin.

**provisioning/datasources/zabbix.yaml**
```yaml
apiVersion: 1

datasources:
  - name: Zabbix
    type: alexanderzobnin-zabbix-datasource
    access: proxy
    url: http://192.168.31.222/api_jsonrpc.php
    isDefault: true
    jsonData:
      username: Admin
    secureJsonData:
      password: zabbix
```
