# Uptime Kuma - Monitorizacion de Servicios

Uptime Kuma es una herramienta self-hosted para monitorizar disponibilidad de servicios y endpoints desde una interfaz web simple.

## Caracteristicas

- Monitorizacion HTTP(s), TCP, Ping, DNS y mas
- Notificaciones por multiples canales (Telegram, Discord, Email, Webhook, etc.)
- Paginas de estado publicas (Status Pages)
- Base de datos SQLite embebida
- Despliegue rapido con Docker Compose

## Requisitos Previos

- Docker Engine
- Docker Compose plugin
- Red Docker externa proxy (si usas proxy inverso)
- Dominio con DNS apuntando al servidor (si publicas por HTTPS)

## Archivos del Repositorio

- compose.yaml: definicion del servicio Uptime Kuma
- README.md: documentacion de despliegue y operacion

Nota: este repo ya no usa archivo .env ni .env.example.

## compose.yaml actual

```yaml
services:
  uptime-kuma:
    container_name: uptime-kuma
    image: louislam/uptime-kuma:2
    restart: unless-stopped
    volumes:
      - ./data:/app/data

# anadir estas lineas al final del archivo para proxy inverso
networks:
  default:
    external: true
    name: proxy
```

## Despliegue rapido

1. Clona el repositorio y entra en la carpeta:

```bash
git clone https://github.com/groales/uptime-kuma.git
cd uptime-kuma
```

2. Crea la red proxy si todavia no existe:

```bash
docker network create proxy
```

3. Inicia el servicio:

```bash
docker compose up -d
```

4. Verifica estado y logs:

```bash
docker compose ps
docker compose logs -f uptime-kuma
```

La primera inicializacion suele tardar entre 10 y 20 segundos.

## Acceso

- Si usas proxy inverso: accede a tu dominio, por ejemplo https://uptime.example.com
- Si haces acceso directo temporal (sin proxy): publica puerto 3001 en compose y accede a http://IP_SERVIDOR:3001

En el primer acceso debes crear la cuenta administradora.

## Configuracion recomendada de proxy inverso

Para Nginx Proxy Manager (o equivalente):

- Forward Hostname/IP: uptime-kuma
- Forward Port: 3001
- Websockets Support: habilitado (obligatorio)
- SSL: certificado valido y Force SSL habilitado

## Backup y restauracion

Los datos persisten en ./data, incluyendo la base SQLite de Uptime Kuma.

Backup simple:

```bash
# Desde la carpeta del proyecto
mkdir -p backup
tar -czf backup/uptime-kuma-data-$(date +%Y%m%d-%H%M%S).tar.gz data
```

Restauracion:

```bash
docker compose down
rm -rf data
tar -xzf backup/uptime-kuma-data-FECHA.tar.gz
docker compose up -d
```

## Actualizacion

```bash
docker compose pull uptime-kuma
docker compose up -d uptime-kuma
docker compose logs -f uptime-kuma
```

## Solucion de Problemas

Contenedor no inicia:

```bash
docker compose logs --tail 200 uptime-kuma
```

No carga por dominio:

- Verifica DNS
- Verifica que el proxy enruta a uptime-kuma:3001
- Verifica que WebSockets este habilitado

Permisos en data:

- Asegura permisos de lectura/escritura en la carpeta data del host

## Variables de entorno

No hay variables de entorno requeridas en la configuracion actual del repositorio.

## Recursos

- Documentacion oficial del proyecto: https://github.com/louislam/uptime-kuma/wiki
- Docker Hub: https://hub.docker.com/r/louislam/uptime-kuma
- Issues oficiales: https://github.com/louislam/uptime-kuma/issues

## Licencia

Este repositorio de configuracion: MIT
Uptime Kuma: MIT License
