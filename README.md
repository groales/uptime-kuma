# Uptime Kuma - Monitorización de Servicios

Uptime Kuma es una herramienta de monitorización self-hosted moderna y elegante. Permite monitorizar servicios HTTP(s), TCP, Ping, DNS, Docker containers, y mucho más con notificaciones multi-canal.

## Características

- ✅ **Monitorización multi-protocolo**: HTTP(s), TCP, HTTP(s) Keyword, HTTP(s) Json Query, Ping, DNS, Docker, Steam Game Server
- ✅ **Notificaciones**: 90+ servicios (Telegram, Discord, Slack, Email, Gotify, Webhooks, etc.)
- ✅ **Status Pages**: Páginas de estado públicas personalizables
- ✅ **Mapeo de servicios**: Vista gráfica de dependencias
- ✅ **Certificados SSL**: Monitorización de expiración
- ✅ **Multi-idioma**: Interfaz en múltiples idiomas incluyendo español
- ✅ **2FA**: Autenticación de dos factores
- ✅ **Sin base de datos externa**: Usa SQLite embebida

## Requisitos Previos

- Docker y Docker Compose instalados
- **Proxy inverso configurado**:
  - **Traefik** con red `proxy` externa, o
  - **Nginx Proxy Manager**
- Dominio con DNS apuntando al servidor

⚠️ **IMPORTANTE**: Este repositorio **NO incluye modo standalone**. Requiere proxy inverso (Traefik o NPM) para funcionar.

## Archivos de este Repositorio

Este repositorio contiene archivos de ejemplo:
- `docker-compose.yml` - Configuración base del contenedor
- `.env.example` - Plantilla de variables de entorno
- `docker-compose.override.traefik.yml.example` - Labels para Traefik
- `README.md` - Esta documentación

> 💡 **Tip**: Puedes copiar estos archivos manualmente o clonar el repositorio.

---

## Despliegue con Docker Compose

### 1. Crear Directorio y Archivos

```bash
# Crear directorio
mkdir uptime-kuma
cd uptime-kuma
```

**Opción A: Copiar archivos manualmente** (recomendado)

Copia el contenido de los archivos del repositorio a tu directorio local.

**Opción B: Clonar repositorio**

```bash
git clone https://git.ictiberia.com/groales/uptime-kuma.git
cd uptime-kuma
```

### 2. Elegir Modo de Despliegue

El docker-compose.yml base es:

```yaml
services:
  uptime-kuma:
    container_name: uptime-kuma
    image: louislam/uptime-kuma:2
    restart: unless-stopped
    volumes:
      - uptime-kuma_data:/app/data

volumes:
  uptime-kuma_data:
    name: uptime-kuma_data

# añadir estas líneas al final del archivo para proxy inverso 
networks:
  default:
    external: true
    name: proxy
```

**Opción A: Traefik** (con SSL automático)

```bash
cp docker-compose.override.traefik.yml.example docker-compose.override.yml
cp .env.example .env
nano .env  # Editar: configurar DOMAIN_HOST
```

**Opción B: Nginx Proxy Manager**

```bash
# No requiere .env ni override
# Configuración manual en NPM después del despliegue
```

### 3. Iniciar el Servicio

```bash
docker compose up -d
```

La inicialización tarda **10-20 segundos** (creación de base de datos SQLite).

> 💡 **Tip**: Usa `docker compose logs -f uptime-kuma` para ver el progreso en tiempo real.

### 4. Verificar el Despliegue

```bash
# Ver logs en tiempo real
docker compose logs -f uptime-kuma

# Verificar estado del contenedor
docker compose ps

# Ver base de datos creada
docker compose exec uptime-kuma ls -lh /app/data/
```

### 5. Acceder al Servicio

**Con Traefik**: Accede directamente a `https://<DOMAIN_HOST>`
- Ejemplo: `https://uptime.example.com`
- SSL configurado automáticamente

**Con NPM**: Configura el proxy host en la interfaz de NPM
- Forward Hostname: `uptime-kuma`
- Forward Port: `3001`
- ✅ Activa **Websockets Support** (obligatorio)
- Configura SSL

> ⚠️ **Primera configuración**: Al acceder por primera vez, crea tu cuenta de administrador.

---

## Configuración Detallada por Modo

### Traefik (SSL automático con Let's Encrypt)

**Requisitos**:
- Stack de Traefik desplegado
- Red `proxy` creada
- DNS apuntando al servidor

**Configuración**:

1. Copiar archivo override:
   ```bash
   cp docker-compose.override.traefik.yml.example docker-compose.override.yml
   ```

2. Configurar `.env`:
   ```bash
   cp .env.example .env
   nano .env
   ```

3. Editar `DOMAIN_HOST`:
   ```env
   DOMAIN_HOST=uptime.example.com
   ```

4. Desplegar:
   ```bash
   docker compose up -d
   ```

**Acceso**: `https://uptime.example.com`

---

### Nginx Proxy Manager (NPM)

**Requisitos**:
- NPM desplegado
- Ambos servicios en red `proxy`

**Configuración en NPM**:

1. Ir a **Hosts** → **Proxy Hosts** → **Add Proxy Host**

2. **Details**:
   - **Domain Names**: `uptime.example.com`
   - **Scheme**: `http`
   - **Forward Hostname/IP**: `uptime-kuma`
   - **Forward Port**: `3001`
   - ✅ **Block Common Exploits**
   - ✅ **Websockets Support** (⚠️ **IMPORTANTE**: Uptime Kuma requiere WebSockets)

3. **SSL**:
   - ✅ **SSL Certificate**: Request a new SSL Certificate
   - ✅ **Force SSL**
   - ✅ **HTTP/2 Support**
   - ✅ **HSTS Enabled**

4. **Save**

**Acceso**: `https://uptime.example.com`

---

## Comparación: Traefik vs NPM

| Característica | Traefik | Nginx Proxy Manager |
|---|---|---|
| **Configuración** | Archivo `.env` | Interfaz web |
| **SSL** | Automático (Let's Encrypt) | Manual (1 click) |
| **WebSockets** | Automático | Requiere activar checkbox |
| **Renovación SSL** | Automática | Automática |
| **Complejidad** | Media | Baja |

---

## Configuración Inicial

### 1. Crear cuenta administrador

Al acceder por primera vez:

1. Ingresa **username** (será el usuario de login)
2. Ingresa **password** (mínimo 6 caracteres)
3. Click en **Create**

⚠️ **IMPORTANTE**: No hay usuario por defecto. El primer usuario que crees será administrador.

### 2. Configurar idioma

1. Click en el icono de perfil (esquina superior derecha)
2. **Settings** → **Appearance**
3. **Language**: Selecciona **Español** o tu idioma preferido

### 3. Configurar notificaciones (recomendado)

1. **Settings** → **Notifications**
2. Click en **Setup Notification**
3. Elige el tipo de notificación:

   **Telegram**:
   - Obtener bot token de [@BotFather](https://t.me/botfather)
   - Obtener Chat ID de [@userinfobot](https://t.me/userinfobot)
   - Pegar ambos en Uptime Kuma

   **Discord**:
   - Crear Webhook en canal de Discord
   - Pegar URL del webhook

   **Email (SMTP)**:
   - Hostname: `smtp.gmail.com` (ejemplo)
   - Port: `587`
   - Username: tu email
   - Password: app password
   - From Email: tu email
   - To Email: destinatario

4. Click en **Test** para verificar
5. **Save**

### 4. Habilitar autenticación 2FA (recomendado)

1. **Settings** → **Security**
2. **Two Factor Authentication**
3. Escanear QR con app (Google Authenticator, Authy)
4. Ingresar código de 6 dígitos
5. **Enable**

---

## Añadir Monitorización

### Monitor HTTP(s)

1. Click en **Add New Monitor**
2. **Monitor Type**: HTTP(s)
3. Configurar:
   - **Friendly Name**: Nombre descriptivo (ej: "Web Producción")
   - **URL**: `https://example.com`
   - **Heartbeat Interval**: `60` segundos (frecuencia de check)
   - **Retries**: `3` (reintentos antes de marcar como down)
   - **Heartbeat Retry Interval**: `60` segundos
   - **Advanced**: 
     - **Method**: `GET` (o `POST`, `HEAD`)
     - **Expected Status Code**: `200-299` (códigos de éxito)
     - **Keyword**: Buscar palabra clave en respuesta (opcional)
4. **Notifications**: Seleccionar notificaciones configuradas
5. **Save**

### Monitor TCP

Para monitorizar puertos (SSH, bases de datos, etc.):

1. **Monitor Type**: TCP Port
2. Configurar:
   - **Friendly Name**: "SSH Server"
   - **Hostname**: `192.168.1.100`
   - **Port**: `22`
   - **Heartbeat Interval**: `60` segundos
3. **Save**

### Monitor Ping

Para monitorizar disponibilidad de red:

1. **Monitor Type**: Ping
2. Configurar:
   - **Friendly Name**: "Router Principal"
   - **Hostname**: `192.168.1.1`
   - **Heartbeat Interval**: `60` segundos
3. **Save**

### Monitor Docker Container

Para monitorizar contenedores Docker:

1. **Monitor Type**: Docker Container
2. Configurar:
   - **Friendly Name**: "NetBox Container"
   - **Docker Daemon**: `unix:///var/run/docker.sock` (requiere montar socket)
   - **Container Name**: `netbox`
3. **Save**

⚠️ **Nota**: Requiere añadir volumen en `docker-compose.yml`:
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro
```

### Monitor DNS

Para verificar resolución DNS:

1. **Monitor Type**: DNS
2. Configurar:
   - **Friendly Name**: "DNS Cloudflare"
   - **Hostname**: `example.com`
   - **Resolver Server**: `1.1.1.1`
   - **Resource Record Type**: `A`
3. **Save**

---

## Status Pages (Páginas de Estado)

### Crear página pública

1. Click en **Status Pages** (menú lateral)
2. Click en **Add New Status Page**
3. Configurar:
   - **Title**: Nombre de la página (ej: "Estado de Servicios")
   - **Slug**: URL amigable (ej: `servicios`)
   - **Description**: Descripción opcional
   - **Theme**: Light/Dark
   - **Custom CSS**: Personalización avanzada (opcional)
4. **Add Group**: Agrupar monitores (ej: "Aplicaciones", "Infraestructura")
5. Arrastrar monitores a los grupos
6. **Save**

**Acceso público**: `https://uptime.example.com/status/servicios`

### Tipos de visualización

- **Default**: Lista vertical con estado actual
- **List**: Lista compacta
- **Badge**: Badges estilo shields.io
- **Grid**: Cuadrícula

---

## Backup y Restauración

### Backup Manual

Uptime Kuma usa SQLite embebida en `/app/data/kuma.db`.

**Método 1: Copia de volumen**

```bash
# Detener contenedor
docker compose stop uptime-kuma

# Copiar base de datos
docker compose cp uptime-kuma:/app/data/kuma.db ./backup/kuma-$(date +%Y%m%d).db

# Reiniciar contenedor
docker compose start uptime-kuma
```

**Método 2: Backup en caliente (requiere Litestream - avanzado)**

Instalar Litestream para backups continuos a S3/MinIO.

### Backup Automático con Script

```bash
#!/bin/bash
# backup-uptime-kuma.sh

BACKUP_DIR="/backups/uptime-kuma"
RETENTION_DAYS=30

mkdir -p $BACKUP_DIR

# Backup
docker compose -f /path/to/uptime-kuma/docker-compose.yml \
  exec -T uptime-kuma \
  sqlite3 /app/data/kuma.db ".backup /app/data/kuma-backup.db"

docker compose -f /path/to/uptime-kuma/docker-compose.yml \
  cp uptime-kuma:/app/data/kuma-backup.db \
  $BACKUP_DIR/kuma-$(date +%Y%m%d-%H%M%S).db

# Limpiar backups antiguos
find $BACKUP_DIR -name "kuma-*.db" -mtime +$RETENTION_DAYS -delete

echo "Backup completado: $(date)"
```

**Cron**: Ejecutar diariamente a las 2 AM
```cron
0 2 * * * /usr/local/bin/backup-uptime-kuma.sh >> /var/log/backup-uptime-kuma.log 2>&1
```

### Restauración

```bash
# Detener contenedor
docker compose stop uptime-kuma

# Restaurar base de datos
docker compose cp ./backup/kuma-20241205.db uptime-kuma:/app/data/kuma.db

# Reiniciar contenedor
docker compose start uptime-kuma
```

---

## Actualización

### Actualizar a última versión

Uptime Kuma usa versionado semántico. Tag `2` siempre apunta a la última versión 2.x.

```bash
# Backup previo
docker compose exec uptime-kuma sqlite3 /app/data/kuma.db ".backup /app/data/kuma-pre-update.db"

# Actualizar imagen
docker compose pull uptime-kuma

# Recrear contenedor
docker compose up -d uptime-kuma

# Verificar logs
docker compose logs -f uptime-kuma
```

### Actualización automática con Watchtower

Añadir en `docker-compose.yml`:

```yaml
services:
  watchtower:
    image: containrrr/watchtower:latest
    container_name: watchtower-uptime-kuma
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_INCLUDE_STOPPED=false
      - WATCHTOWER_MONITOR_ONLY=false
      - WATCHTOWER_SCHEDULE=0 0 4 * * *  # 4 AM diario
    command: uptime-kuma
```

---

## Solución de Problemas

### El contenedor no inicia

**Verificar logs**:
```bash
docker compose logs uptime-kuma
```

**Problemas comunes**:
- Base de datos corrupta → Restaurar desde backup
- Permisos en volumen → `chown -R 1000:1000 /var/lib/docker/volumes/uptime-kuma_data`

### No se puede acceder a la interfaz

**Traefik**:
```bash
# Verificar labels
docker inspect uptime-kuma | grep -A 10 Labels

# Verificar red proxy
docker network inspect proxy
```

**NPM**:
- Verificar WebSockets habilitado
- Verificar Forward Port: `3001`

### Notificaciones no funcionan

**Telegram**:
- Verificar bot token válido
- Verificar chat ID correcto (debe comenzar con `-`)
- Bot debe estar añadido al grupo/canal

**Email**:
- Verificar SMTP port: `587` (TLS) o `465` (SSL)
- Gmail requiere "App Password" (no contraseña de cuenta)
- Verificar firewall no bloquea puertos SMTP

### Monitores reportan "down" incorrectamente

**Ajustar configuración**:
- Aumentar **Heartbeat Interval** (ej: de 60 a 120 segundos)
- Aumentar **Retries** (ej: de 3 a 5 intentos)
- Aumentar **Heartbeat Retry Interval**
- Verificar **Expected Status Code** (algunos sitios usan 301/302)

**Verificar desde contenedor**:
```bash
# Probar conectividad desde el contenedor
docker compose exec uptime-kuma wget -O- https://example.com

# Probar DNS
docker compose exec uptime-kuma nslookup example.com
```

### Base de datos corrupta

**Síntomas**: Error al iniciar, monitores desaparecen

**Solución**:
```bash
# Intentar reparar
docker compose exec uptime-kuma sqlite3 /app/data/kuma.db "PRAGMA integrity_check;"

# Si falla, restaurar desde backup
docker compose stop uptime-kuma
docker compose cp ./backup/kuma-latest.db uptime-kuma:/app/data/kuma.db
docker compose start uptime-kuma
```

### Status Page no carga

- Verificar slug correcto en URL
- Verificar página configurada como "Public"
- Limpiar cache del navegador (Ctrl+F5)

---

## Variables de Entorno

Uptime Kuma tiene configuración mínima por variables de entorno (la mayoría se configura desde UI).

| Variable | Descripción | Ejemplo | Requerida |
|----------|-------------|---------|-----------|
| `DOMAIN_HOST` | Dominio para Traefik | `uptime.example.com` | Sí (Traefik) |

**Configuración avanzada** (opcional):

```yaml
environment:
  - UPTIME_KUMA_PORT=3001  # Cambiar puerto (por defecto 3001)
  - UPTIME_KUMA_HOST=0.0.0.0  # Bind address
```

---

## Recursos

- 📖 [Documentación oficial](https://github.com/louislam/uptime-kuma/wiki)
- 🐛 [Reportar problemas](https://github.com/louislam/uptime-kuma/issues)
- 💬 [Comunidad Discord](https://discord.gg/uptime-kuma)
- 🐳 [Docker Hub](https://hub.docker.com/r/louislam/uptime-kuma)

---

## Licencia

Este repositorio de configuración: MIT  
Uptime Kuma: [MIT License](https://github.com/louislam/uptime-kuma/blob/master/LICENSE)
