# Documentación Técnica: WhatsApp API Manager

## 1. Introducción

El WhatsApp API Manager es una aplicación web desarrollada con Node.js y React que proporciona una interfaz para gestionar múltiples sesiones de WhatsApp Web y permite enviar y recibir mensajes programáticamente. Esta solución se basa en la biblioteca whatsapp-web.js y está diseñada para integrarse con otros sistemas, como el plugin de WordPress para WooCommerce.

## 2. Requisitos previos del sistema

### 2.1. Hardware

- Procesador: 2 núcleos como mínimo (4 recomendado)
- RAM: 2GB como mínimo (4GB recomendado)
- Almacenamiento: 20GB disponibles como mínimo
- Conexión a Internet estable

### 2.2. Software

- **Node.js**: Versión 14.x o superior
- **npm**: Versión 6.x o superior
- **MongoDB**: Versión 4.x o superior
- **Docker** (opcional para despliegue containerizado): Versión 19.x o superior
- **Docker Compose** (opcional): Versión 1.29.x o superior

### 2.3. Red

- Puerto 3000 accesible (configurable)
- Acceso a Internet para la conexión con WhatsApp Web
- Posibilidad de crear túneles WebSocket

## 3. Estructura del proyecto

El proyecto sigue una arquitectura completa de aplicación web con componentes frontend y backend:

```
react_template/
├── backend/                # Código del servidor Node.js
│   ├── config/             # Configuración del servidor
│   ├── controllers/        # Controladores de la API
│   ├── middleware/         # Middleware de Express
│   ├── models/             # Modelos de datos y lógica de WhatsApp
│   ├── services/           # Servicios de autenticación, sesiones, etc.
│   ├── utils/              # Utilidades generales
│   └── server.js           # Punto de entrada del servidor
├── src/                    # Código del frontend React
│   ├── components/         # Componentes de React
│   ├── context/            # Contexto de React
│   ├── hooks/              # Hooks personalizados
│   ├── pages/              # Páginas de la aplicación
│   ├── services/           # Servicios del frontend
│   ├── App.jsx             # Componente principal
│   └── main.jsx            # Punto de entrada del frontend
├── public/                 # Archivos estáticos
├── build/                  # Output de producción (generado)
├── whatsapp-sessions/      # Almacenamiento de sesiones WhatsApp
├── package.json            # Dependencias y scripts
├── vite.config.js          # Configuración de Vite
├── .env                    # Variables de entorno
├── Dockerfile              # Configuración de Docker
└── docker-compose.yml      # Configuración de Docker Compose
```

## 4. Instrucciones de instalación paso a paso

### 4.1. Instalación local (desarrollo)

1. **Clonar el repositorio** (suponiendo que existe un repositorio Git)

```bash
git clone [URL_DEL_REPOSITORIO]
cd WhatsApp-API-Manager
```

2. **Instalar dependencias**

```bash
npm install
```

3. **Configurar MongoDB**

Debes tener una instancia de MongoDB accesible. Puedes instalarla localmente o usar un servicio en la nube como MongoDB Atlas.

4. **Crear archivo de variables de entorno**

Crea un archivo `.env` en la raíz del proyecto con el siguiente contenido:

```
# Configuración del servidor
PORT=3000
NODE_ENV=development

# MongoDB
MONGODB_URI=mongodb://localhost:27017/whatsapp-api

# Configuración JWT
JWT_SECRET=TuClaveSecretaSuperSegura

# WhatsApp API
WA_CLIENT_ID=your-client-id
WA_CLIENT_SECRET=your-client-secret

# CORS
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
```

5. **Iniciar el servidor en modo desarrollo**

```bash
npm run dev
```

Esto iniciará tanto el servidor backend (en el puerto 3000) como el servidor de desarrollo de Vite para el frontend React (típicamente en el puerto 5173).

### 4.2. Instalación con Docker

1. **Clonar el repositorio**

```bash
git clone [URL_DEL_REPOSITORIO]
cd WhatsApp-API-Manager
```

2. **Configurar variables de entorno**

Crea un archivo `.env` con la configuración necesaria como se mostró anteriormente, pero asegúrate de que la URL de MongoDB apunte al servicio de MongoDB dentro de Docker si lo estás usando:

```
MONGODB_URI=mongodb://mongodb:27017/whatsapp-api
```

3. **Construir e iniciar los contenedores**

```bash
docker-compose up -d
```

Esto construirá la imagen de Docker y ejecutará los contenedores en segundo plano.

## 5. Configuración de variables de entorno y archivos de configuración

### 5.1. Variables de entorno principales

| Variable | Descripción | Valor por defecto |
|----------|-------------|------------------|
| `PORT` | Puerto en el que se ejecutará el servidor | `3000` |
| `NODE_ENV` | Entorno de ejecución (development, production) | `development` |
| `MONGODB_URI` | URL de conexión a la base de datos MongoDB | `mongodb://localhost:27017/whatsapp-api` |
| `JWT_SECRET` | Clave secreta para firmar tokens JWT | Ninguno (requerido) |
| `JWT_EXPIRY` | Tiempo de expiración de tokens JWT en segundos | `86400` (24 horas) |
| `WA_CLIENT_ID` | ID de cliente para autenticación externa | Ninguno (opcional) |
| `WA_CLIENT_SECRET` | Secreto de cliente para autenticación externa | Ninguno (opcional) |
| `CORS_ORIGINS` | Orígenes permitidos para CORS, separados por comas | `*` |
| `WP_JWT_PUBLIC_KEY` | Clave pública para validar tokens JWT de WordPress | Ninguno (opcional) |

### 5.2. Configuración del backend

La configuración del backend se encuentra en el directorio `backend/config/`. El archivo principal es `config.js`, que importa las variables de entorno y establece valores por defecto.

Para modificar la configuración avanzada, edita este archivo según tus necesidades.

## 6. Proceso de despliegue

### 6.1. Despliegue en entorno de desarrollo

El entorno de desarrollo utiliza Vite para el frontend y nodemon para el backend, lo que permite una recarga automática cuando se realizan cambios en el código:

```bash
npm run dev
```

### 6.2. Despliegue en producción

1. **Construir el frontend**

```bash
npm run build
```

Esto generará los archivos estáticos del frontend en la carpeta `build`.

2. **Iniciar el servidor de producción**

```bash
NODE_ENV=production npm start
```

### 6.3. Despliegue con Docker en producción

1. **Construir la imagen de producción**

```bash
docker build -t whatsapp-api-manager:production .
```

2. **Desplegar con Docker Compose**

Asegúrate de que el archivo `.env` está configurado para producción y ejecuta:

```bash
docker-compose up -d
```

### 6.4. Configuración de proxy inverso (recomendado)

En producción, se recomienda configurar un proxy inverso como Nginx o Apache para gestionar el tráfico HTTPS y redirigirlo al puerto de la aplicación:

**Ejemplo de configuración con Nginx:**

```nginx
server {
    listen 80;
    server_name whatsapp-api.tudominio.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name whatsapp-api.tudominio.com;

    ssl_certificate /etc/letsencrypt/live/whatsapp-api.tudominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/whatsapp-api.tudominio.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 7. Descripción de endpoints principales y su uso

### 7.1. Autenticación

#### Login con usuario y contraseña

- **URL**: `POST /api/auth/login`
- **Cuerpo**:
  ```json
  {
    "username": "admin",
    "password": "password123"
  }
  ```
- **Respuesta exitosa**:
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "507f1f77bcf86cd799439011",
      "username": "admin",
      "roles": ["admin"]
    }
  }
  ```

#### Autenticación con token de WordPress

- **URL**: `POST /api/auth/wordpress`
- **Cuerpo**:
  ```json
  {
    "wpToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
  ```
- **Respuesta exitosa**: Similar a la de login

#### Verificar token

- **URL**: `GET /api/auth/verify`
- **Cabeceras**: 
  ```
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  ```
- **Respuesta exitosa**:
  ```json
  {
    "id": "507f1f77bcf86cd799439011",
    "username": "admin",
    "roles": ["admin"]
  }
  ```

### 7.2. Gestión de Sesiones

#### Listar sesiones

- **URL**: `GET /api/sessions`
- **Cabeceras**: Authorization (JWT)
- **Respuesta exitosa**:
  ```json
  [
    {
      "clientId": "session1",
      "status": "CONNECTED",
      "createdAt": "2023-01-01T12:00:00.000Z",
      "updatedAt": "2023-01-01T12:10:00.000Z"
    },
    {
      "clientId": "session2",
      "status": "DISCONNECTED",
      "createdAt": "2023-01-02T12:00:00.000Z",
      "updatedAt": "2023-01-02T12:30:00.000Z"
    }
  ]
  ```

#### Crear sesión

- **URL**: `POST /api/sessions`
- **Cabeceras**: Authorization (JWT)
- **Cuerpo**:
  ```json
  {
    "clientId": "nueva_session",
    "config": {
      "restartOnLogout": true
    }
  }
  ```
- **Respuesta exitosa**:
  ```json
  {
    "clientId": "nueva_session",
    "status": "INITIALIZING",
    "createdAt": "2023-01-03T12:00:00.000Z",
    "updatedAt": "2023-01-03T12:00:00.000Z"
  }
  ```

#### Obtener estado de sesión

- **URL**: `GET /api/sessions/:clientId/status`
- **Cabeceras**: Authorization (JWT)
- **Respuesta exitosa**:
  ```json
  {
    "clientId": "session1",
    "status": "CONNECTED",
    "updatedAt": "2023-01-03T12:05:00.000Z"
  }
  ```

#### Obtener código QR para autenticación

- **URL**: `GET /api/sessions/:clientId/qr`
- **Cabeceras**: Authorization (JWT)
- **Respuesta exitosa**:
  ```json
  {
    "clientId": "session1",
    "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...",
    "status": "QR_READY"
  }
  ```

#### Eliminar sesión

- **URL**: `DELETE /api/sessions/:clientId`
- **Cabeceras**: Authorization (JWT)
- **Respuesta exitosa**:
  ```json
  {
    "success": true
  }
  ```

### 7.3. Mensajes

#### Enviar mensaje

- **URL**: `POST /api/messages/:clientId/send`
- **Cabeceras**: Authorization (JWT)
- **Cuerpo**:
  ```json
  {
    "chatId": "34600000000@c.us",
    "message": "Hola, este es un mensaje de prueba",
    "options": {}
  }
  ```
- **Respuesta exitosa**:
  ```json
  {
    "messageId": "true_34600000000@c.us_3EB0645D8B0D95C17A"
  }
  ```

#### Obtener historial de mensajes

- **URL**: `GET /api/messages/:clientId/:chatId/history?limit=50&before=messageId`
- **Cabeceras**: Authorization (JWT)
- **Respuesta exitosa**:
  ```json
  [
    {
      "id": "false_34600000000@c.us_3EB0645D8B0D95C17A",
      "body": "Hola, ¿cómo estás?",
      "from": "34600000000@c.us",
      "to": "34600000001@c.us",
      "timestamp": 1642424242
    },
    {
      "id": "true_34600000001@c.us_3EB0645D8B0D95C17B",
      "body": "Muy bien, gracias",
      "from": "34600000001@c.us",
      "to": "34600000000@c.us",
      "timestamp": 1642424300
    }
  ]
  ```

## 8. Comunicación en tiempo real (WebSockets)

El API Manager incluye un sistema de WebSockets para recibir actualizaciones en tiempo real sobre eventos de WhatsApp.

### 8.1. Conexión al servidor WebSocket

```javascript
import { io } from "socket.io-client";

const socket = io("https://tu-servidor.com", {
  auth: {
    token: "tu_token_jwt"
  }
});
```

### 8.2. Eventos disponibles

| Evento | Descripción | Datos |
|--------|-------------|-------|
| `session_status` | Actualización del estado de una sesión | `{ clientId, status }` |
| `session_qr` | Nuevo código QR disponible | `{ clientId, qrCode }` |
| `message_received` | Mensaje recibido | `{ clientId, message }` |
| `session_created` | Sesión creada | `{ clientId }` |
| `session_deleted` | Sesión eliminada | `{ clientId }` |

### 8.3. Ejemplo de uso

```javascript
// Escuchar cambios de estado de sesión
socket.on("session_status", (data) => {
  console.log(`La sesión ${data.clientId} ahora está ${data.status}`);
});

// Escuchar nuevos mensajes
socket.on("message_received", (data) => {
  console.log(`Nuevo mensaje de ${data.message.from}:`, data.message.body);
});
```

## 9. Monitoreo y mantenimiento del sistema

### 9.1. Logs del sistema

La aplicación utiliza un sistema de logging interno para registrar eventos importantes:

- Los logs se escriben en la consola en desarrollo
- En producción, se recomienda configurar un sistema de gestión de logs externo

Para verificar los logs en una instancia Docker:

```bash
docker logs whatsapp-api-manager
```

### 9.2. Monitoreo de sesiones

Las sesiones de WhatsApp deben monitorearse regularmente:

1. Comprobar el estado de conexión
2. Verificar que los códigos QR se regeneran correctamente
3. Monitorear el uso de recursos (CPU, memoria)

### 9.3. Respaldo de datos

Se recomienda realizar respaldos regulares de:

1. La base de datos MongoDB
2. El directorio de sesiones de WhatsApp (`whatsapp-sessions/`)

```bash
# Respaldo de MongoDB
mongodump --uri="mongodb://localhost:27017/whatsapp-api" --out=backup_$(date +%Y%m%d)

# Respaldo de sesiones
tar -czf whatsapp_sessions_$(date +%Y%m%d).tar.gz whatsapp-sessions/
```

### 9.4. Actualización del sistema

Para actualizar el sistema a una nueva versión:

1. Detener el servicio actual
2. Respaldar datos y configuración
3. Descargar la nueva versión
4. Fusionar cambios de configuración si es necesario
5. Reconstruir la imagen de Docker (si aplica)
6. Reiniciar el servicio

## 10. Solución de problemas comunes

### 10.1. Problemas de conexión con WhatsApp

**Problema**: No se puede generar o escanear código QR

**Solución**:
- Verificar que Puppeteer puede ejecutarse correctamente
- Si se usa Docker, asegurarse de que la imagen incluye las dependencias necesarias
- Intentar reiniciar la sesión con `DELETE /api/sessions/:clientId` seguido de `POST /api/sessions`

**Problema**: Sesión desconectada frecuentemente

**Solución**:
- Verificar conexión a Internet estable
- Comprobar que el teléfono asociado tiene batería y conexión a Internet
- Configurar `restartOnLogout: true` al crear la sesión

### 10.2. Problemas de autenticación

**Problema**: Token JWT inválido

**Solución**:
- Verificar que `JWT_SECRET` coincide con el utilizado para generar el token
- Comprobar que el token no ha expirado
- Para tokens de WordPress, verificar que `WP_JWT_PUBLIC_KEY` está configurado correctamente

**Problema**: No se pueden validar tokens de WordPress

**Solución**:
- Verificar que la clave pública de WordPress es correcta y está en formato PEM
- Comprobar que el token incluye los claims necesarios (sub, username, roles)

### 10.3. Problemas de rendimiento

**Problema**: Alto consumo de memoria

**Solución**:
- Limitar el número de sesiones concurrentes
- Aumentar la RAM disponible para el contenedor
- Configurar límites de memoria para Node.js: `NODE_OPTIONS="--max-old-space-size=4096"`

**Problema**: Lentitud en respuestas de la API

**Solución**:
- Verificar la conexión a MongoDB (añadir índices si es necesario)
- Comprobar la carga del sistema (CPU, disco, red)
- Implementar un sistema de caché para respuestas frecuentes

## 11. Integración con WordPress

### 11.1. Configuración en WordPress

El API Manager está diseñado para integrarse con el plugin de WordPress para WooCommerce. Para configurar la integración:

1. En el plugin de WordPress, configurar la URL del API Manager
2. Generar un par de claves JWT en WordPress
3. Configurar la clave pública de WordPress en el API Manager mediante la variable de entorno `WP_JWT_PUBLIC_KEY`

### 11.2. Flujo de autenticación

1. WordPress genera un token JWT con información del usuario
2. El frontend envía este token al API Manager mediante `POST /api/auth/wordpress`
3. El API Manager valida el token con la clave pública de WordPress
4. Se genera un nuevo token JWT del API Manager
5. El cliente utiliza este nuevo token para las llamadas a la API

## 12. Consideraciones de seguridad

### 12.1. Protección de tokens JWT

- Usar claves secretas seguras y aleatorias
- Configurar tiempos de expiración adecuados
- Almacenar tokens de forma segura en el cliente (localStorage no es recomendado)

### 12.2. Seguridad en la comunicación

- Utilizar HTTPS para todas las conexiones
- Configurar CORS adecuadamente para limitar orígenes
- Usar WebSockets seguros (WSS)

### 12.3. Protección de sesiones de WhatsApp

- Limitar el acceso a las credenciales de sesión
- Implementar control de acceso basado en roles
- Monitorear actividad sospechosa

## 13. Recursos adicionales

- [Documentación oficial de whatsapp-web.js](https://wwebjs.dev/)
- [Guía de Express.js](https://expressjs.com/es/guide/routing.html)
- [Documentación de Socket.IO](https://socket.io/docs/v4/)
- [Guía de MongoDB](https://docs.mongodb.com/manual/)
- [Documentación de Docker](https://docs.docker.com/)

---

Esta documentación está sujeta a actualizaciones. Última revisión: Junio 2024.