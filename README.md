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
    *   Autenticación básica.
*   **Personalización**: Posibilidad de modificar CSS, añadir bots y apps del Marketplace gratuito.
*   **Omnicanal Básico**: Integración básica con widgets de LiveChat para sitios web.

### Ventajas de esta implementación
*   **Portabilidad**: Todo el entorno está contenerizado; fácil de mover entre servidores.
*   **Persistencia**: Los datos de la base de datos se guardan en el volumen `mongodb_data`.
*   **Escalabilidad Vertical**: Puedes aumentar recursos de tu servidor sin reinstalar.
*   **Aislamiento**: Las dependencias no ensucian tu sistema operativo anfitrión.

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
