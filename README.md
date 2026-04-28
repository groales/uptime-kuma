# Uptime Kuma

Uptime Kuma es una herramienta self-hosted para monitorizar disponibilidad de servicios y endpoints desde una interfaz web simple.

Referencia oficial de instalación: https://github.com/louislam/uptime-kuma/wiki

## Características

- Monitorización HTTP(S), TCP, Ping y DNS.
- Notificaciones por múltiples canales.
- Páginas de estado públicas.
- Persistencia local con SQLite.

## Requisitos Previos

- Docker Engine instalado.
- Docker Compose instalado.
- Red Docker externa `proxy` creada si usarás proxy inverso.

## Archivos de este Repositorio

- `compose.yaml` - Definición del servicio Uptime Kuma.
- `README.md` - Esta documentación.

Nota: este repositorio no usa `.env` ni `.env.example`.

---

## Despliegue con Docker Compose

### 1. Clonar el repositorio

```bash
git clone https://github.com/groales/uptime-kuma.git
cd uptime-kuma
```

### 2. Revisar `compose.yaml`

```yaml
services:
  uptime-kuma:
    container_name: uptime-kuma
    image: louislam/uptime-kuma:2
    restart: unless-stopped
    volumes:
      - ./data:/app/data

networks:
  default:
    external: true
    name: proxy
```

### 3. Levantar el servicio

```bash
docker network create proxy
docker compose up -d
```

---

## Método Alternativo: Crear Manualmente

Puedes copiar el `compose.yaml` en una carpeta nueva y ejecutar el mismo despliegue.

---

## Acceso Inicial

- Con proxy inverso: accede al dominio configurado.
- Con acceso directo temporal: publica puerto `3001` y accede a `http://IP_SERVIDOR:3001`.

En el primer acceso debes crear la cuenta administradora.

## Comandos Útiles

```bash
docker compose ps
docker compose logs -f uptime-kuma
docker compose restart uptime-kuma
docker compose pull uptime-kuma
docker compose up -d uptime-kuma
docker compose down
```

## Estructura de Volúmenes

```text
Bind mount:
└── ./data -> /app/data
```

## Configuración Avanzada

Para Nginx Proxy Manager (o equivalente):

- Forward Hostname/IP: `uptime-kuma`
- Forward Port: `3001`
- WebSockets: habilitado
- SSL: certificado válido y Force SSL habilitado

## Solución de Problemas

Si el contenedor no inicia:

```bash
docker compose logs --tail 200 uptime-kuma
```

Si no carga por dominio:

- Verifica DNS.
- Verifica que el proxy enruta a `uptime-kuma:3001`.
- Verifica WebSockets habilitado.

## Seguridad

- Usa HTTPS si expones el servicio.
- Restringe acceso al panel con firewall/VPN si aplica.
- Usa contraseña fuerte para la cuenta administradora.

## Backup y Restauración

```bash
# Backup
mkdir -p backup
tar -czf backup/uptime-kuma-data-$(date +%Y%m%d-%H%M%S).tar.gz data

# Restauración
docker compose down
rm -rf data
tar -xzf backup/uptime-kuma-data-FECHA.tar.gz
docker compose up -d
```

## Actualización

```bash
docker compose pull uptime-kuma
docker compose up -d uptime-kuma
docker compose logs -f uptime-kuma
```

## Recursos

- Documentación oficial: https://github.com/louislam/uptime-kuma/wiki
- Docker Hub: https://hub.docker.com/r/louislam/uptime-kuma
- Issues oficiales: https://github.com/louislam/uptime-kuma/issues

## Licencia

Este repositorio de configuración es de uso libre. Uptime Kuma se distribuye bajo licencia MIT.
