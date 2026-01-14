# Rocket.Chat - Docker Self-Hosted Environment

Esta configuración despliega una instancia de **Rocket.Chat** utilizando Docker y Docker Compose, pre-configurada para una puesta en marcha rápida y eficiente.

## 🚀 Versiones Utilizadas
*   **Rocket.Chat**: `8.0.1`
*   **MongoDB**: `7.0.28`
*   **NodeJS**: `22.16.0` (Incluido en la imagen de Rocket.Chat)
*   **Platform**: `linux`

## ✨ Características de esta Configuración (Automated Setup)
Esta instalación ha sido personalizada para automatizar pasos tediosos:
1.  **Usuario Administrador Automático**: No necesitas registrar manualmente la primera cuenta.
    *   **Usuario**: `admin`
    *   **Contraseña**: `password` (¡Cámbiala inmediatamente!)
    *   **Email**: `admin@example.com`
2.  **Omitir Asistente (Setup Wizard)**: El sistema arranca directamente en la interfaz de chat, saltando los pasos iniciales de configuración.
3.  **Réplica Set de MongoDB Automatizada**: Incluye un servicio `mongodb-init` que configura automáticamente el replica set necesario para las funcionalidades de tiempo real (Oplog).

## 🆓 Rocket.Chat Community Edition (Versión Gratuita)
Al usar la imagen oficial `rocket.chat:8.0.1`, por defecto accedes a las características de la versión comunitaria/gratis.

### ¿Qué ofrece la versión GRATIS?
*   **Usuarios Ilimitados**: A diferencia de soluciones SaaS, no pagas por asiento (limitado solo por tu hardware).
*   **Historial de Mensajes Ilimitado**: Acceso total a todas las conversaciones pasadas.
*   **Control Total de Datos**: Tus datos residen en tu servidor, no en la nube de un tercero.
*   **Funcionalidades Core**:
    *   Canales públicos y privados.
    *   Mensajes directos y discusiones.
    *   Compartir archivos.
    *   Videoconferencias (vía integración Jitsi/Pexip).
    *   **Integración Active Directory / LDAP (Core)**:
        *   **Autenticación**: Login con credenciales de dominio (Soporte OpenLDAP y Active Directory).
        *   **Sincronización**: Mapeo básico de datos (Nombre, Email, UID) y sincronización de Avatares.
        *   **Control de Acceso**: Filtros de búsqueda (Search Filter) para restringir el login a grupos específicos.
        *   **Seguridad**: Comunicación encriptada nativa (SSL/LDAPS y StartTLS).
        *   **Gestión**: Merge automático de usuarios por email y fallback a login local si falla el AD.
    *   Autenticación básica (Email/Password).
*   **Personalización**: Posibilidad de modificar CSS, añadir bots y apps del Marketplace gratuito.
*   **Omnicanal Básico**: Integración básica con widgets de LiveChat para sitios web.

### 🔌 Configuración de Active Directory / LDAP (Paso a Paso)
Para conectar tu servidor con un dominio (AD/LDAP), sigue estos pasos dentro de la administración:

1.  Ve a **Administration > Workspace > Settings > LDAP**.
2.  **Connection**:
    *   **Enable**: `True`
    *   **Host**: `tu-servidor-ldap.com` (o IP)
    *   **Port**: `389` (LDAP) o `636` (LDAPS)
    *   **Reconnect**: `True` (Reconectar automáticamente si cae la conexión)
3.  **Authentication**:
    *   **User DN**: Usuario de servicio para leer el directorio (ej: `cn=Manager,dc=ejemplo,dc=com` o `dominio\usuario`).
    *   **Password**: Contraseña del usuario de servicio.
4.  **Encryption** (Seguridad):
    *   Si usas puerto 636, selecciona `SSL/LDAPS`.
    *   Si usas puerto 389 pero quieres seguridad, selecciona `StartTLS`.
    *   Nota: Si usas certificados autofirmados, marca `True` en "Reject Unauthorized" solo si has importado el CA, de lo contrario `False` (usar con precaución en prod).
5.  **User Search** (Filtros):
    *   **Base DN**: Dónde buscar usuarios (ej: `ou=Usuarios,dc=empresa,dc=com`).
    *   **Filter**: Filtro para encontrar usuarios válidos.
        *   AD: `(&(sAMAccountName=#{username})(memberOf=cn=RocketUsers,ou=Groups,dc=empresa,dc=com))`
        *   OpenLDAP: `(uid=#{username})`
    *   **Search Field**: `sAMAccountName` (AD) o `uid` (OpenLDAP).
6.  **Data Sync** (Sincronización):
    *   Mapea los campos para que Rocket.Chat obtenga los datos del AD:
    *   **Username Field**: `sAMAccountName` o `uid`
    *   **Email Field**: `mail`
    *   **Name Field**: `displayName` o `cn`
    *   **Sync User Data on Login**: `True` (Actualiza datos al loguear).
    *   **Merge Existing Users**: `True` (Si el email ya existe en Rocket.Chat, lo fusiona con el usuario de AD).

> 💡 **Tip**: Usa el botón "Test Connection" para verificar la conectividad antes de guardar.

### ⚠️ Diferencias con la versión Enterprise (De Pago)
Lo explicado arriba **SÍ está incluido en la versión Gratis**. Sin embargo, ten en cuenta estas limitaciones:
*   **Sincronización en segundo plano**: La versión gratis solo actualiza datos cuando el usuario se loguea ("Sync on Login"). La Enterprise permite sincronización periódica automática en segundo plano.
*   **Mapeo de Roles**: No puedes asignar roles (Admin, Moderador) automáticamente según grupos de AD en la versión gratis.
*   **Gestión de Equipos**: La sincronización de "Teams" de Rocket.Chat con grupos de AD es una función Enterprise.

### Ventajas de esta implementación
*   **Portabilidad**: Todo el entorno está contenerizado; fácil de mover entre servidores.
*   **Persistencia**: Los datos de la base de datos se guardan en el volumen `mongodb_data`.
*   **Escalabilidad Vertical**: Puedes aumentar recursos de tu servidor sin reinstalar.
*   **Aislamiento**: Las dependencias no ensucian tu sistema operativo anfitrión.

### 📦 Gestión de Backups (Base de Datos)
Sí, estamos utilizando un **volumen de Docker** llamado `mongodb_data` para persistir los datos. Esto significa que aunque borres el contenedor, tus datos siguen seguros en el disco de tu servidor.

#### 1. Crear un Respaldo (Backup)
Para generar un archivo de respaldo completo de tu base de datos (chat, usuarios, configuraciones) sin detener el servicio:

```bash
# Ejecuta este comando en la misma carpeta donde está tu docker-compose.yaml
docker compose exec mongodb bash -c 'mongodump --archive --gzip' > backup_rocketchat_$(date +%F).gz
```
*   Esto creará un archivo llamado algo como `backup_rocketchat_2024-01-14.gz` en tu carpeta actual.
*   Guarda este archivo en un lugar seguro (otro servidor, nube, disco externo).

#### 2. Restaurar un Respaldo
⚠️ **Advertencia**: Esto sobrescribirá los datos actuales.

1.  Detén Rocket.Chat (opcional pero recomendado para evitar inconsistencias):
    ```bash
    docker compose stop rocketchat
    ```
2.  Ejecuta el comando de restauración:
    ```bash
    # Reemplaza 'NOMBRE_ARCHIVO.gz' con tu respaldo real
    docker compose exec -T mongodb bash -c 'mongorestore --archive --gzip --drop' < backup_rocketchat_2024-01-14.gz
    ```
3.  Reinicia los servicios:
    ```bash
    docker compose start rocketchat
    ```

#### 3. ¿Dónde están mis datos físicamente?
Como usamos un volumen de Docker, los datos "crudos" suelen estar en:
`/var/lib/docker/volumes/rocketchat_mongodb_data/_data` (La ruta exacta depende del nombre de tu carpeta del proyecto).

___
## 📋 Requisitos del Sistema
El hardware necesario depende directamente de la cantidad de usuarios activos.

| Recurso | Mínimo (1-50 usuarios) | Recomendado (50-200 usuarios) | Alto Rendimiento (200-500 usuarios) | Empresarial (500+ usuarios) |
| :--- | :--- | :--- | :--- | :--- |
| **CPU** | 1 Core (2.0 GHz+) | 2 Cores | 4 Cores (3.0 GHz+) | 4-8 Cores+ (High Freq) |
| **RAM** | 2 GB | 4 GB | 8 GB | 16 GB - 32 GB |
| **Disco** | 20 GB (SSD) | 50 GB+ | 100 GB+ (SSD NVMe) | 200 GB+ (Recomendado S3) |
| **OS** | Linux | Linux | Linux (Optimized) | Linux (Cluster/K8s) |

## 🛠️ Cómo Iniciar

1.  Asegúrate de tener el archivo `.env` configurado (puedes copiar `.env.example`).
2.  Ejecuta el siguiente comando:

```bash
docker-compose up -d --build
```

3.  Espera unos minutos a que los servicios inicien completamente.
4.  Accede a `http://localhost:3000`.
5.  Loguéate con:
    *   User: `admin`
    *   Pass: `password`
