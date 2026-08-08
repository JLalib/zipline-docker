# 🚀 Zipline Docker - Servidor de Subida de Archivos y Acortador Profesional

[![GitHub Stars](https://img.shields.io/github/stars/diced/zipline?style=for-the-badge&logo=github)](https://github.com/diced/zipline)
[![Docker Pulls](https://img.shields.io/docker/pulls/diced/zipline?style=for-the-badge&logo=docker)](https://hub.docker.com/r/diced/zipline)
[![License](https://img.shields.io/github/license/diced/zipline?style=for-the-badge)](https://github.com/diced/zipline/blob/main/LICENSE)

## 📋 Descripción general

**Zipline** es un servidor de subida de archivos diseñado específicamente para ser el backend ideal de herramientas como **ShareX**, pero que funciona como una plataforma completa de compartición de archivos y acortador de URLs. Es la solución perfecta para quienes quieren dejar de depender de Imgur o Dropbox y tener sus propios assets bajo control total.

Combina la potencia de un servidor de archivos con la utilidad de un acortador de enlaces. Todo en una sola interfaz moderna, rápida y extremadamente fácil de configurar mediante Docker.

## ✨ Características principales

- 🎯 **Backend para ShareX** - Diseñado para integrarse a la perfección con ShareX, facilitando la subida instantánea de capturas
- 🔗 **Acortador de URLs integrado** - No solo subas archivos; crea enlaces cortos y personalizados para cualquier destino
- 🤖 **Webhooks de Discord** - Notifica automáticamente a tus canales de Discord cada vez que se suba un archivo específico
- 🔐 **Seguridad avanzada** - Soporta Passkeys, 2FA y OAuth2 (Discord, GitHub, Google) para que solo tú y tus invitados accedan a los datos
- 🛡️ **Protección de archivos** - Protege archivos sensibles con contraseñas y gestiona cuotas de subida por usuario
- ⚡ **Optimización de media** - Compresión inteligente de imágenes y generación de thumbnails para vídeos automáticamente
- 📁 **Organización total** - Usa carpetas y etiquetas para clasificar miles de archivos sin perder el control
- 🔌 **API robusta** - Crea tus propias integraciones vía API
- 📱 **PWA** - Aplicación web progresiva para acceso rápido desde el móvil

## 📋 Requisitos del sistema

- ✅ **CPU con soporte AVX** (Obligatorio: Zipline no funciona en CPUs sin AVX)
- 🐳 **Docker** y **Docker Compose**
- 🐘 **PostgreSQL 16** (Incluido en el compose)
- 💾 **RAM**: 1-2 GB recomendados
- 💿 **Espacio en disco**: Depende de la cantidad de archivos que planees alojar
- 🌐 **Navegador moderno**

> ⚠️ **Aviso Crítico**: Asegúrate de que tu procesador soporte la instrucción AVX. Si usas algunos procesadores ARM muy antiguos o Celeron básicos, es posible que la imagen de Docker no inicie.

## 🐳 Instalación

### Paso 1: Generar secretos y crear `.env`

Zipline requiere un `CORE_SECRET` y una contraseña de base de datos. Ejecuta esto en tu terminal:

```bash
echo "POSTGRESQL_PASSWORD=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" > .env
echo "CORE_SECRET=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" >> .env
```

### Paso 2: Crear `docker-compose.yml`

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

🌐 **http://localhost:3000** - Dashboard de Zipline

## ⚙️ Configuración

1. **Variables de entorno obligatorias** - `POSTGRESQL_PASSWORD` y `CORE_SECRET` en el archivo `.env`
2. **Base de datos** - PostgreSQL 16 se configura automáticamente vía `DATABASE_URL`
3. **Volúmenes persistentes** - `uploads`, `public` y `themes` mapeados localmente
4. **Healthcheck** - PostgreSQL verifica disponibilidad antes de iniciar Zipline
5. **Puertos** - Por defecto expone el puerto 3000 (configurable en `docker-compose.yml`)

## 🚀 Primeros pasos

1. **Configuración inicial** - Entra en la web y crea tu cuenta de administrador. Configura el `DATASOURCE_LOCAL_DIRECTORY` si quieres cambiar dónde se guardan los archivos físicamente.

2. **Integrar con ShareX** - En Zipline, ve a la sección de API/Configuración y obtén tu clave de API. En ShareX → Destinos → Custom uploader settings. Configura la URL de subida de Zipline y añade la API Key en los headers.

3. **Crear tu primer enlace corto** - Ve a la sección de acortador en el dashboard. Introduce la URL de destino y elige un alias personalizado. ¡Listo! Ya tienes un link corto propio.

4. **Configurar OAuth2** - Crea una aplicación en el portal de desarrolladores de Discord o Google. Copia el Client ID y Secret en la configuración de Zipline. Ahora tus usuarios podrán entrar con un solo clic.

## 💡 Casos de uso

- 📸 **Backend de capturas** - Sube tus screenshots de ShareX instantáneamente sin usar servicios externos
- 🔗 **Acortador privado** - Gestiona tus propios enlaces cortos para redes sociales o mensajes sin tracking
- 📦 **Repositorio de assets** - Guarda imágenes y vídeos para tus proyectos web con acceso rápido vía URL
- 🔐 **Compartición segura** - Envía archivos confidenciales con contraseñas y fecha de caducidad
- 🤖 **Notificaciones automáticas** - Configura webhooks para que tu servidor de Discord avise cada vez que alguien suba un archivo

## 🔒 Acceso remoto seguro

### HTTPS con Caddy (producción)

**Caddyfile**
```caddyfile
zipline.tudominio.com {
    reverse_proxy localhost:3000
}
```

Acceso remoto seguro: **https://zipline.tudominio.com** con SSL automático y cifrado de extremo a extremo.

## 🛠️ Gestión y mantenimiento

| Acción | Comando |
|--------|---------|
| **Ver logs en tiempo real** | `docker compose logs -f zipline` |
| **Backup de la base de datos** | `docker compose exec postgresql pg_dump -U zipline zipline > zipline_db_backup.sql` |
| **Actualizar la versión** | `docker compose pull && docker compose up -d` |
| **Monitorear consumo** | `docker stats zipline postgresql` |

## 📝 Licencia

Este proyecto está licenciado bajo **AGPL-3.0** - ver el archivo [LICENSE](https://github.com/diced/zipline/blob/main/LICENSE) para más detalles.

---

> 📖 **Guía completa**: [Cómo instalar Zipline en Docker - Servidor de subida de archivos y acortador profesional](https://genbyte.blogspot.com/2026/07/como-instalar-zipline-en-docker.html)