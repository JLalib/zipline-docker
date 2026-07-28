# 📤 Zipline: Servidor de subida de archivos y acortador de URLs con Docker

[![GitHub](https://img.shields.io/badge/GitHub-Repositorio-blue)](https://github.com/JLalib/zipline-docker) [![Docker](https://img.shields.io/badge/Docker-Zipline-blue)](https://hub.docker.com/r/diced/zipline) [![License](https://img.shields.io/badge/Licencia-MIT-green)](https://github.com/JLalib/zipline-docker/blob/main/LICENSE)

## 📋 Descripción general

Zipline es un servidor de subida de archivos diseñado específicamente para ser el backend ideal de herramientas como ShareX, pero que funciona como una plataforma completa de compartición de archivos y acortador de URLs. Es la solución perfecta para quienes quieren dejar de depender de Imgur o Dropbox y tener sus propios assets bajo control total.

Este repositorio contiene la configuración necesaria para desplegar Zipline con Docker Compose, siguiendo el tutorial de Genbyte para montar tu propio servidor de subida de archivos y acortador de enlaces.

## ✨ Características principales

- **Backend para ShareX**: diseñado para integrarse a la perfección con ShareX, facilitando la subida instantánea de capturas
- **Acortador de URLs integrado**: crea enlaces cortos y personalizados para cualquier destino
- **Webhooks de Discord**: notifica automáticamente a tus canales de Discord cada vez que se suba un archivo
- **Seguridad avanzada**: soporte para Passkeys, 2FA y OAuth2 (Discord, GitHub, Google)
- **Optimización de media**: compresión inteligente de imágenes y generación de thumbnails para vídeos
- **Organización total**: carpetas y etiquetas para clasificar miles de archivos sin perder el control
- **Control de acceso**: protege archivos sensibles con contraseñas y gestiona cuotas de subida por usuario
- **API robusta + PWA**: crea tus propias integraciones vía API o usa la app web progresiva en tu móvil

## 📋 Requisitos del sistema

- **CPU con soporte AVX** (obligatorio: Zipline no funciona en CPUs sin AVX)
- Docker y Docker Compose instalados
- PostgreSQL 16 (incluido en el compose)
- Al menos 1-2 GB de RAM recomendados
- Espacio en disco según la cantidad de archivos que planees alojar
- Navegador moderno

⚠️ **Aviso crítico**: asegúrate de que tu procesador soporte la instrucción AVX. Si usas procesadores ARM muy antiguos o Celeron básicos, es posible que la imagen de Docker no inicie.

## 🐳 Instalación

### Paso 1: Generar secretos y crear .env

Zipline requiere un `CORE_SECRET` y una contraseña de base de datos. Ejecuta esto en tu terminal:

```bash
echo "POSTGRESQL_PASSWORD=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" > .env
echo "CORE_SECRET=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" >> .env
```

💡 También puedes usar el script `generate-secrets.sh` incluido en este repositorio: `bash generate-secrets.sh`

### Paso 2: Crear docker-compose.yml

Crea un archivo `docker-compose.yml` con el siguiente contenido:

```yaml
services:
  postgresql:
    image: postgres:16
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - POSTGRES_USER=zipline
      - POSTGRES_DB=zipline
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD', 'pg_isready', '-U', 'zipline']
      interval: 10s
      timeout: 5s
      retries: 5

  zipline:
    image: ghcr.io/diced/zipline
    ports:
      - "3000:3000"
    env_file:
      - .env
    environment:
      - DATABASE_URL=postgres://zipline:${POSTGRESQL_PASSWORD}@postgresql:5432/zipline
    depends_on:
      postgresql:
        condition: service_healthy
    volumes:
      - ./uploads:/zipline/uploads
      - ./public:/zipline/public
      - ./themes:/zipline/themes
    restart: unless-stopped

volumes:
  pgdata:
```

### Paso 3: Iniciar el servidor

```bash
docker compose up -d
```

### Acceder

`http://localhost:3000` - Dashboard de Zipline

## ⚙️ Configuración

Antes de iniciar los contenedores, ten en cuenta:

1. **POSTGRESQL_PASSWORD**: contraseña generada aleatoriamente para el usuario `zipline` de PostgreSQL
2. **CORE_SECRET**: clave de cifrado interna de Zipline, imprescindible y generada aleatoriamente
3. **DATASOURCE_LOCAL_DIRECTORY**: variable opcional si quieres cambiar dónde se guardan los archivos físicamente
4. Verifica que tu CPU soporta AVX antes de desplegar, o el contenedor no arrancará

💡 Consejo: nunca reutilices el mismo `CORE_SECRET` entre instancias distintas de Zipline.

## 🚀 Primeros pasos

1. Asegúrate de que tu CPU soporta AVX y de tener Docker y Docker Compose instalados
2. Genera el `.env` con los secretos aleatorios (Paso 1 de la instalación)
3. Crea el `docker-compose.yml` y ejecuta `docker compose up -d`
4. Abre tu navegador en `http://localhost:3000` y crea tu cuenta de administrador
5. Integra Zipline con ShareX:
   - En Zipline, ve a la sección de API/Configuración y obtén tu clave de API
   - En ShareX → Destinos → Custom uploader settings
   - Configura la URL de subida de Zipline y añade la API Key en los headers
6. Crea tu primer enlace corto desde la sección de acortador del dashboard
7. Configura OAuth2 (opcional):
   - Crea una aplicación en el portal de desarrolladores de Discord o Google
   - Copia el Client ID y Secret en la configuración de Zipline

## 💡 Casos de uso

- **Backend de capturas**: sube tus screenshots de ShareX instantáneamente sin usar servicios externos
- **Acortador privado**: gestiona tus propios enlaces cortos para redes sociales o mensajes sin tracking
- **Repositorio de assets**: guarda imágenes y vídeos para tus proyectos web con acceso rápido vía URL
- **Compartición segura**: envía archivos confidenciales con contraseñas y fecha de caducidad
- **Notificaciones automáticas**: configura webhooks para que tu servidor de Discord avise de cada subida

## 🔒 Acceso remoto seguro (opcional)

Si deseas acceder a Zipline desde fuera de tu red local de forma segura, puedes usar un proxy inverso como Caddy, Nginx Proxy Manager o Traefik para obtener un certificado gratuito de Let's Encrypt.

### Configuración Caddyfile (ejemplo)

```
zipline.tudominio.com {
    reverse_proxy localhost:3000
}
```

### Resultado

Acceso mediante `https://zipline.tudominio.com` con SSL automático y cifrado de extremo a extremo.

## 🛠️ Gestión y mantenimiento

### Ver logs en tiempo real

```bash
docker compose logs -f zipline
```

### Backup de la base de datos

```bash
docker compose exec postgresql pg_dump -U zipline zipline > zipline_db_backup.sql
```

### Actualizar la versión

```bash
docker compose pull
docker compose up -d
```

### Monitorear consumo

```bash
docker stats zipline postgresql
```

## 📝 Licencia

Este proyecto se basa en [Zipline](https://github.com/diced/zipline), licenciado bajo AGPL-3.0. La configuración y documentación proporcionada aquí está bajo la [MIT License](https://github.com/JLalib/zipline-docker/blob/main/LICENSE).

---

> ✨ **Nota**: Este repositorio contiene la configuración Docker y documentación extraída del tutorial de Genbyte: [Cómo instalar Zipline en Docker - Servidor de subida de archivos y acortador profesional](https://genbyte.blogspot.com/2026/07/como-instalar-zipline-en-docker.html)
