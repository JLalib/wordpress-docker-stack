# 🐳 WordPress Docker Stack

[![GitHub](https://img.shields.io/badge/GitHub-Repo-blue?logo=github)](https://github.com/genbyte/wordpress-docker-stack)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue?logo=docker)](https://hub.docker.com/r/library/wordpress)
[![License](https://img.shields.io/badge/License-GPL%20v3-green)](LICENSE)

## 📋 Descripción general

**WordPress en Docker** es una instalación completa del CMS más popular del mundo (40% de internet) empaquetada en contenedores con **MariaDB** y **phpMyAdmin**, permitiendo tener un sitio web totalmente funcional y autohospedado sin dependencia de hosting SaaS como WordPress.com o Wix.

Es la solución ideal para blogs, sitios corporativos, tiendas online con **total control y propiedad de datos**. Stack production-ready con Nginx reverse proxy, volúmenes persistentes, backups triviales y HTTPS automático vía Caddy/Let's Encrypt.

## ✨ Características principales

- 📦 **WordPress core última versión** - CMS más popular, 40% de internet, potente, flexible, robusto
- 🗄️ **MariaDB 10.6+ optimizada** - Fork MySQL compatible, ligera, rápida, eficiente
- 🖥️ **phpMyAdmin interfaz web** - Gestión BD visual, backup/restore sin CLI
- ⚡ **Nginx high-performance** - Web server rápido, bajo recursos, reverse proxy integrado
- 💾 **Volúmenes persistentes** - Datos seguros en Docker volumes, backups triviales (cp de volumes)
- 🔒 **HTTPS automático** - SSL certificados gratis Let's Encrypt via Caddy integration
- 🔌 **60K+ plugins + 10K+ temas** - Ecosistema masivo, extensibilidad total, personalización infinita
- 🌐 **Multi-sitio WordPress** - Múltiples blogs en una instalación, gestión centralizada
- 👥 **Users + Roles RBAC** - Control acceso granular: Admin, Editor, Author, Contributor, Subscriber
- 🖼️ **Media library optimizado** - Imágenes, videos, archivos con redimensionamiento automático
- 📈 **SEO-friendly** - Yoast SEO plugin, sitemaps, canonicals, structured data
- 🚀 **Production-ready** - Tested, escalable, seguro, usado en millones de sitios
- ⚙️ **Cache plugins soportados** - Redis, Memcached via compose
- 🛠️ **Debugging tools** - WP-CLI, database optimization tools, performance monitoring
- 🛡️ **Seguridad hardened** - WP hardening plugins, multiidioma, WPML soporte
- 🧪 **Staging/development environments** - Fácil creación de entornos de prueba

## 📋 Requisitos del sistema

- ✅ **Docker** instalado y funcionando
- ✅ **Docker Compose** (versión 20+ recomendada, plugin `docker compose`)
- ✅ **1 GB - 4 GB RAM** mínimo (depende tráfico sitio: 2 GB para tráfico pequeño, 4+ GB para medio)
- ✅ **5 GB - 50 GB espacio disco** (para DB + uploads + contenido)
- ✅ **Puerto 80 y 443** disponibles (HTTP y HTTPS, configurables)
- ✅ **Volúmenes persistentes** para WordPress, MariaDB, phpMyAdmin
- ✅ **Dominio apuntando a servidor** (para HTTPS automático con Caddy)
- ✅ **Email servidor SMTP** (opcional, para notificaciones WordPress)
- ✅ **Navegador moderno** (editar posts, gestión admin)
- 💡 **Setup recomendado**: 2 GB RAM mínimo, SSD para mejor rendimiento

## 🐳 Instalación

### Opción 1: Docker Compose completo (recomendado)

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  # MariaDB - Base de datos
  mariadb:
    image: mariadb:latest
    container_name: wordpress_db
    restart: unless-stopped
    environment:
      - MYSQL_DATABASE=wordpress
      - MYSQL_ROOT_PASSWORD=tu_contraseña_root_fuerte
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=tu_contraseña_wordpress_fuerte
      - TZ=Europe/Madrid
    volumes:
      - db_data:/var/lib/mysql
    command: --default-authentication-plugin=mysql_native_password

  # WordPress - CMS
  wordpress:
    image: wordpress:php8.2-fpm
    container_name: wordpress_app
    restart: unless-stopped
    depends_on:
      - mariadb
    environment:
      - WORDPRESS_DB_HOST=mariadb:3306
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=tu_contraseña_wordpress_fuerte
      - WORDPRESS_DB_NAME=wordpress
      - WORDPRESS_TABLE_PREFIX=wp_
      - WORDPRESS_DEBUG=false
    volumes:
      - wordpress_data:/var/www/html

  # Nginx - Reverse Proxy / Web Server
  nginx:
    image: nginx:alpine
    container_name: wordpress_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - wordpress_data:/var/www/html:ro
    depends_on:
      - wordpress

  # phpMyAdmin - Gestión BD web UI
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: wordpress_phpmyadmin
    restart: unless-stopped
    environment:
      - PMA_HOST=mariadb
      - PMA_USER=root
      - PMA_PASSWORD=tu_contraseña_root_fuerte
    ports:
      - "8080:80"

volumes:
  wordpress_data:
  db_data:
EOF
```

```bash
# Crear nginx.conf (si no existe)
cat > nginx.conf << 'EOF'
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 100M;

    upstream wordpress {
        server wordpress:9000;
    }

    server {
        listen 80;
        server_name _;
        root /var/www/html;
        index index.php index.html;

        location ~ \.php$ {
            fastcgi_pass wordpress;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include /etc/nginx/fastcgi_params;
        }

        location ~ /\.ht {
            deny all;
        }
    }
}
EOF
```

```bash
# Levantar stack
docker compose up -d
```

### Acceder (setup WordPress)

| Servicio | URL | Credenciales |
|----------|-----|--------------|
| **WordPress Installer** | `http://localhost` | Asistente 5-minutos |
| **phpMyAdmin** | `http://localhost:8080` | user: `root` / pass: `tu_contraseña_root_fuerte` |

## ⚙️ Configuración

1. **Cambiar contraseñas por defecto** - Edita `docker-compose.yml` y reemplaza `tu_contraseña_root_fuerte` y `tu_contraseña_wordpress_fuerte` por contraseñas seguras únicas
2. **Configurar zona horaria** - Ajusta `TZ=Europe/Madrid` a tu zona (ej: `America/Mexico_City`, `UTC`)
3. **Personalizar puertos** - Modifica `"80:80"`, `"443:443"`, `"8080:80"` si hay conflictos
4. **Ajustar límites PHP** - En `nginx.conf`: `client_max_body_size 100M` para subidas grandes
5. **Configurar dominio para HTTPS** - Ver sección [Acceso remoto seguro](#-acceso-remoto-seguro)

## 🚀 Primeros pasos

1. **Login admin WordPress** - Abre `http://localhost/wp-admin`, ingresa usuario y contraseña creados en el instalador
2. **Crear primer post** - Dashboard → Posts → Add New → Título, contenido, categorías, etiquetas → Publish
3. **Personalizar sitio (apariencia)** - Dashboard → Appearance → Themes → Browse free themes o sube custom theme (ZIP) → Activate → Customize
4. **Agregar plugins (extensiones)** - Dashboard → Plugins → Add New → Busca plugin (ej: "Yoast SEO") → Install → Activate
5. **Gestionar usuarios y roles** - Dashboard → Users → Add New → Email, usuario, contraseña, rol (Administrator, Editor, Author, Contributor, Subscriber)
6. **Configurar páginas estáticas** - Dashboard → Pages → Add New → Crea About, Contact, Terms → Appearance → Menus → crear menú
7. **Configurar inicio de sesión del sitio** - Dashboard → Settings → General → Site Title, Tagline, Site URL, Timezone, date format
8. **Backup de BD con phpMyAdmin** - Abre `http://localhost:8080` → Database: wordpress → Export → Descarga SQL file
9. **Backup de archivos WordPress** - `docker cp wordpress_app:/var/www/html ./wordpress-backup-$(date +%Y%m%d)`
10. **Instalar Yoast SEO** - Dashboard → Plugins → Add New → Busca "Yoast SEO" → Install → Activate → Configura meta descriptions
11. **Habilitar permanentes (Pretty URLs)** - Dashboard → Settings → Permalinks → Elige "Post name" → Guarda
12. **Configurar comentarios y discusiones** - Dashboard → Settings → Discussion → Habilita/deshabilita comentarios, moderación, Akismet

## 💡 Casos de uso

- 📝 **Blogs personales** - Full control, sin publicidad de WordPress.com, datos tuyos
- 🏢 **Sitios corporativos** - Presencia web profesional, portfolio, contacto
- 🛒 **Tiendas online** - WooCommerce plugin integrado, ventas completas, inventory management
- 🎨 **Agencias digitales** - Sitios clientes self-hosted, mantenimiento fácil
- 📸 **Portfolios creativos** - Fotógrafos, artistas, diseñadores, galería de trabajos
- 🔄 **Alternativa WordPress.com/Wix** - Total control, bajo costo, open source, no vendor lock-in

## 🔒 Acceso remoto seguro

### HTTPS con Caddy (producción)

```bash
# Caddyfile para WordPress
cat > Caddyfile << 'EOF'
midominio.com www.midominio.com {
    reverse_proxy localhost:80
}
EOF

# Ejecutar Caddy (Docker)
docker run -d \
  --name caddy \
  --restart unless-stopped \
  -p 80:80 -p 443:443 \
  -v $(pwd)/Caddyfile:/etc/caddy/Caddyfile \
  -v caddy_data:/data \
  -v caddy_config:/config \
  caddy:latest
```

**Acceso:** `https://midominio.com` con HTTPS automático y certificado Let's Encrypt

### IMPORTANTE: Configurar WordPress URL

```bash
# Editar wp-config.php en WordPress container
docker exec wordpress_app bash -c "nano /var/www/html/wp-config.php"

# Agregar o modificar estas líneas:
define('WP_HOME', 'https://midominio.com');
define('WP_SITEURL', 'https://midominio.com');
define('FORCE_SSL_ADMIN', true);
```

## 🛠️ Gestión y mantenimiento

### Ver logs

```bash
# WordPress
docker logs -f wordpress_app

# MariaDB
docker logs -f wordpress_db

# Nginx
docker logs -f wordpress_nginx
```

### Backup completo (BD + archivos)

```bash
mkdir -p ./backups

# Backup base datos
docker exec wordpress_db mysqldump -u wordpress -p tu_contraseña_wordpress wordpress > ./backups/wordpress-db-$(date +%Y%m%d).sql

# Backup archivos WordPress
docker cp wordpress_app:/var/www/html ./backups/wordpress-files-$(date +%Y%m%d)
```

### Restore de backup

```bash
# Base de datos
docker exec -i wordpress_db mysql -u wordpress -p tu_contraseña wordpress < ./backups/wordpress-db-YYYYMMDD.sql

# Archivos: reemplaza /var/www/html en volume o copia archivos
docker cp ./backups/wordpress-files-YYYYMMDD/. wordpress_app:/var/www/html/
```

### Reiniciar servicios

```bash
# Todos
docker compose restart

# Servicio específico
docker compose restart wordpress
```

### Actualizar WordPress, plugins, temas

```bash
# Vía Dashboard WordPress → Updates → Click Update automáticamente

# O vía WP-CLI en container:
docker exec wordpress_app wp core update --allow-root
docker exec wordpress_app wp plugin update --all --allow-root
docker exec wordpress_app wp theme update --all --allow-root
```

### Limpiar caché y optimizar BD

```bash
# Optimizar tablas MariaDB
docker exec wordpress_db mysqlcheck -u wordpress -p tu_contraseña --optimize --all-databases

# O instalar plugin "WP-Optimize" para limpieza automática
```

### Monitorear consumo

```bash
docker stats wordpress_app wordpress_db wordpress_nginx
```

### Aumentar límite upload archivos

```bash
# Editar wp-config.php
docker exec wordpress_app bash -c "nano /var/www/html/wp-config.php"

# Agregar:
define('WP_MEMORY_LIMIT', '256M');
define('WP_MAX_MEMORY_LIMIT', '512M');
```

## 📝 Licencia

Este proyecto está licenciado bajo **GPL v3** - ver archivo [LICENSE](LICENSE) para detalles.

WordPress, MariaDB y phpMyAdmin son software open source bajo sus respectivas licencias (GPL/AGPL).

---

> 📖 **Basado en el tutorial:** [Cómo instalar WordPress en Docker - Stack completo con MariaDB y phpMyAdmin autohospedado](https://genbyte.blogspot.com/2026/08/como-instalar-wordpress-en-docker-stack.html)
>
> 🎥 **Canal YouTube:** [Genbyte](https://youtube.com/@genbyte) | 📧 **Newsletter:** [Suscríbete](https://genbyte.blogspot.com) | ☕ **Apoya:** [Ko-fi](https://ko-fi.com/genbyte)