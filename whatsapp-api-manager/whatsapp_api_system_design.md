# Sistema de API No Oficial para WhatsApp - Diseño de Sistema

## Implementación y enfoque

Basándonos en el análisis previo, este sistema proporcionará una API RESTful que permitirá a múltiples clientes interactuar con WhatsApp a través de un único backend centralizado basado en la librería whatsapp-web.js. El sistema gestionará múltiples sesiones de WhatsApp para distintos usuarios/clientes, cada uno con su propia autenticación y niveles de acceso.

### Componentes críticos que abordaremos:

1. **Middleware de autenticación JWT**: Implementación segura para validar y autorizar solicitudes a la API.
2. **Endpoint de monitoreo de sesiones**: Para supervisar el estado de las conexiones de WhatsApp.
3. **Mecanismo de recuperación de sesiones**: Para mantener la persistencia tras reinicios del servidor.
4. **Sistema de registro específico por cliente**: Para auditoría y seguimiento de operaciones.
5. **Control de acceso basado en roles (RBAC)**: Para limitar funcionalidades según tipo de usuario.
6. **Integración JWT de WordPress**: Para autenticar usuarios desde sistemas WordPress existentes.
7. **Interfaz de gestión frontend**: Panel web para administrar sesiones y configuraciones.

### Tecnologías principales:

- **Backend**: Node.js con Express.js
- **Base de datos**: MongoDB para almacenamiento de configuraciones y metadatos
- **Autenticación**: JWT (JSON Web Tokens)
- **Cliente WhatsApp**: whatsapp-web.js
- **Frontend**: React con Material-UI
- **Comunicación en tiempo real**: Socket.io

## Estructura de datos e interfaces

La estructura de datos y interfaces se detallan en el archivo `whatsapp_api_class_diagram.mermaid`.

## Flujo de programa

El flujo de programa detallado se encuentra en el archivo `whatsapp_api_sequence_diagram.mermaid`.

## Componentes críticos detallados

### 1. Middleware de autenticación JWT

El middleware JWT será responsable de validar todas las solicitudes a la API y proporcionar un sistema robusto de autorización basado en roles.

```javascript
class JWTMiddleware {
  constructor(authManager) {
    this.authManager = authManager;
  }
  
  // Middleware principal para autenticación
  authenticate(req, res, next) {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
      return res.status(401).json({ error: 'Token no proporcionado' });
    }
    
    this.authManager.validateToken(token)
      .then(decoded => {
        req.user = decoded;
        next();
      })
      .catch(err => {
        res.status(401).json({ error: 'Token inválido o expirado' });
      });
  }
  
  // Middleware para comprobación de roles
  roleCheck(requiredRoles) {
    return (req, res, next) => {
      const userRoles = req.user?.roles || [];
      
      const hasRequiredRole = requiredRoles.some(role => userRoles.includes(role));
      
      if (hasRequiredRole) {
        next();
      } else {
        res.status(403).json({ error: 'Acceso denegado: rol insuficiente' });
      }
    };
  }
  
  // Middleware para validar tokens de WordPress
  validateWordPressToken(req, res, next) {
    const wpToken = req.body.wpToken;
    
    if (!wpToken) {
      return res.status(400).json({ error: 'Token de WordPress no proporcionado' });
    }
    
    this.authManager.validateWordPressJWT(wpToken)
      .then(decoded => {
        req.wpUser = decoded;
        next();
      })
      .catch(err => {
        res.status(401).json({ error: 'Token de WordPress inválido' });
      });
  }
}
```

### 2. Endpoint de API para monitoreo de sesiones

Implementación de endpoints para monitorear el estado de las sesiones de WhatsApp:

```javascript
// En ApiController.js
class ApiController {
  constructor(server, authManager) {
    this.server = server;
    this.authManager = authManager;
    this.logger = new Logger();
    this.jwtMiddleware = new JWTMiddleware(authManager);
  }
  
  setupRoutes(app) {
    // Endpoints de sesión
    app.get('/api/sessions', this.jwtMiddleware.authenticate, this.jwtMiddleware.roleCheck(['admin']), this.handleListSessions.bind(this));
    app.get('/api/sessions/:clientId/status', this.jwtMiddleware.authenticate, this.handleGetSessionStatus.bind(this));
    app.post('/api/sessions', this.jwtMiddleware.authenticate, this.handleCreateSession.bind(this));
    app.delete('/api/sessions/:clientId', this.jwtMiddleware.authenticate, this.jwtMiddleware.roleCheck(['admin']), this.handleDeleteSession.bind(this));
    
    // Otros endpoints...
  }
  
  // Manejador para listar sesiones
  async handleListSessions(req, res) {
    try {
      const sessions = await this.server.sessionManager.listSessions();
      res.json(sessions);
    } catch (error) {
      this.logger.error('Error al listar sesiones', { error: error.message });
      res.status(500).json({ error: 'Error al obtener la lista de sesiones' });
    }
  }
  
  // Manejador para obtener estado de sesión
  async handleGetSessionStatus(req, res) {
    const { clientId } = req.params;
    
    // Verificar permisos para acceder a esta sesión específica
    if (!this.authManager.validateRoleAccess(req.user.roles, 'sessions', 'read', clientId)) {
      return res.status(403).json({ error: 'No tienes permiso para acceder a esta sesión' });
    }
    
    try {
      const session = await this.server.sessionManager.getSession(clientId);
      
      if (!session) {
        return res.status(404).json({ error: 'Sesión no encontrada' });
      }
      
      const status = session.getStatus();
      res.json(status);
    } catch (error) {
      this.logger.error(`Error al obtener estado de sesión ${clientId}`, { error: error.message });
      res.status(500).json({ error: 'Error al obtener el estado de la sesión' });
    }
  }
}
```

### 3. Mecanismo de recuperación de sesiones

El mecanismo de recuperación de sesiones garantiza que las sesiones de WhatsApp persistan incluso después de reinicios del servidor:

```javascript
class SessionManager {
  constructor(dbAdapter) {
    this.sessionStore = new Map();
    this.dbAdapter = dbAdapter;
    this.logger = new Logger();
  }
  
  // Crear una nueva sesión
  async createSession(clientId, config) {
    if (this.sessionStore.has(clientId)) {
      throw new Error(`La sesión para el cliente ${clientId} ya existe`);
    }
    
    const session = new ClientSession(clientId, config);
    await session.initialize();
    
    this.sessionStore.set(clientId, session);
    
    // Guardar metadatos de la sesión en la base de datos
    await this.dbAdapter.insertOne('sessions', {
      clientId,
      config,
      createdAt: new Date(),
      status: session.getStatus()
    });
    
    return session;
  }
  
  // Guardar el estado de la sesión
  async saveSessionState(clientId) {
    const session = this.sessionStore.get(clientId);
    
    if (!session) {
      throw new Error(`Sesión no encontrada para el cliente ${clientId}`);
    }
    
    const whatsappClient = session.whatsappClient;
    const sessionData = await whatsappClient.pupPage.evaluate(() => {
      return window.localStorage;
    });
    
    await this.dbAdapter.updateOne(
      'sessionData',
      { clientId },
      { 
        $set: {
          data: sessionData,
          updatedAt: new Date()
        }
      },
      { upsert: true }
    );
    
    return true;
  }
  
  // Cargar todas las sesiones guardadas
  async loadAllSessions() {
    try {
      const sessions = await this.dbAdapter.find('sessions', {});
      
      for (const sessionInfo of sessions) {
        try {
          const sessionData = await this.loadSessionState(sessionInfo.clientId);
          const session = new ClientSession(sessionInfo.clientId, sessionInfo.config);
          
          if (sessionData) {
            await session.initialize();
            await session.restoreSession(sessionData);
          } else {
            await session.initialize();
          }
          
          this.sessionStore.set(sessionInfo.clientId, session);
          this.logger.info(`Sesión restaurada para ${sessionInfo.clientId}`);
        } catch (err) {
          this.logger.error(`Error al restaurar sesión ${sessionInfo.clientId}`, { error: err.message });
        }
      }
      
      return Array.from(this.sessionStore.values());
    } catch (error) {
      this.logger.error('Error al cargar sesiones', { error: error.message });
      throw error;
    }
  }
  
  // Cargar datos de sesión específica
  async loadSessionState(clientId) {
    const sessionData = await this.dbAdapter.findOne('sessionData', { clientId });
    return sessionData?.data;
  }
}
```

### 4. Sistema de registro específico por cliente

Sistema de logging que permite rastrear acciones por cliente e identificar problemas específicos:

```javascript
class Logger {
  constructor(adapter, config = {}) {
    this.adapter = adapter || console;
    this.config = config;
  }
  
  log(level, message, meta = {}) {
    const timestamp = new Date().toISOString();
    const logEntry = {
      timestamp,
      level,
      message,
      ...meta
    };
    
    // Registrar según el adaptador configurado
    if (this.adapter === console) {
      this.adapter[level](`${timestamp} [${level.toUpperCase()}] ${message}`, meta);
    } else {
      this.adapter.log(logEntry);
    }
    
    // Si está configurado, también guardar en base de datos
    if (this.config.persistToDb && meta.clientId) {
      this.persistLog(logEntry, meta.clientId);
    }
  }
  
  info(message, meta) {
    this.log('info', message, meta);
  }
  
  warn(message, meta) {
    this.log('warn', message, meta);
  }
  
  error(message, meta) {
    this.log('error', message, meta);
  }
  
  debug(message, meta) {
    if (this.config.debugEnabled) {
      this.log('debug', message, meta);
    }
  }
  
  // Crear un logger específico para un cliente
  createClientLogger(clientId) {
    const clientLogger = new Logger(this.adapter, this.config);
    
    // Sobreescribir métodos para incluir automáticamente el clientId
    const methods = ['info', 'warn', 'error', 'debug'];
    
    methods.forEach(method => {
      clientLogger[method] = (message, meta = {}) => {
        this[method](message, { ...meta, clientId });
      };
    });
    
    return clientLogger;
  }
  
  // Persistir logs en base de datos
  async persistLog(logEntry, clientId) {
    try {
      const dbAdapter = this.config.dbAdapter;
      await dbAdapter.insertOne('clientLogs', {
        ...logEntry,
        clientId,
        createdAt: new Date()
      });
    } catch (error) {
      console.error('Error al persistir log:', error);
    }
  }
}
```

### 5. Control de acceso basado en roles (RBAC)

Sistema RBAC para gestionar permisos de forma granular:

```javascript
class AuthManager {
  constructor(authConfig, userRepo) {
    this.tokenBlacklist = new Set();
    this.userRepo = userRepo;
    this.authConfig = authConfig;
    this.logger = new Logger();
    
    // Definición básica de roles y permisos
    this.rolePermissions = {
      'admin': {
        'sessions': ['create', 'read', 'update', 'delete', 'list'],
        'messages': ['send', 'read', 'list'],
        'users': ['create', 'read', 'update', 'delete', 'list']
      },
      'user': {
        'sessions': ['create', 'read', 'update'],
        'messages': ['send', 'read']
      },
      'readonly': {
        'sessions': ['read'],
        'messages': ['read']
      }
    };
  }
  
  // Validar un token JWT
  async validateToken(token) {
    try {
      // Verificar si el token está en la lista negra
      if (this.tokenBlacklist.has(token)) {
        throw new Error('Token revocado');
      }
      
      const decoded = jwt.verify(token, this.authConfig.jwtSecret, {
        algorithms: ['HS256']
      });
      
      // Verificar si el token ha expirado
      const now = Math.floor(Date.now() / 1000);
      if (decoded.exp && decoded.exp < now) {
        throw new Error('Token expirado');
      }
      
      return decoded;
    } catch (error) {
      this.logger.error('Error al validar token', { error: error.message });
      throw error;
    }
  }
  
  // Crear un nuevo token JWT
  async createToken(user) {
    const payload = {
      sub: user.id,
      username: user.username,
      roles: user.roles,
      iat: Math.floor(Date.now() / 1000),
      exp: Math.floor(Date.now() / 1000) + (this.authConfig.tokenExpirySeconds || 3600)
    };
    
    return jwt.sign(payload, this.authConfig.jwtSecret, { algorithm: 'HS256' });
  }
  
  // Revocar un token
  async revokeToken(token) {
    this.tokenBlacklist.add(token);
    
    try {
      // Decodificar sin verificar para obtener exp
      const decoded = jwt.decode(token);
      
      // Programar limpieza del token de la lista negra después de su expiración
      if (decoded && decoded.exp) {
        const expiryMs = decoded.exp * 1000 - Date.now();
        if (expiryMs > 0) {
          setTimeout(() => {
            this.tokenBlacklist.delete(token);
          }, expiryMs + 10000); // 10s extra para compensar posibles diferencias de reloj
        }
      }
    } catch (error) {
      this.logger.error('Error al programar limpieza de token', { error: error.message });
    }
  }
  
  // Validar acceso basado en rol
  validateRoleAccess(roles, resource, action, resourceId = null) {
    if (!Array.isArray(roles)) {
      roles = [roles];
    }
    
    // Verificar si alguno de los roles tiene el permiso necesario
    for (const role of roles) {
      const permissions = this.rolePermissions[role];
      
      if (permissions && 
          permissions[resource] && 
          permissions[resource].includes(action)) {
        return true;
      }
    }
    
    return false;
  }
  
  // Validar token JWT de WordPress
  async validateWordPressJWT(token) {
    try {
      // Obtener la clave pública de WordPress
      const wpPublicKey = this.authConfig.wordpressJwtPublicKey;
      
      if (!wpPublicKey) {
        throw new Error('Clave pública de WordPress no configurada');
      }
      
      const decoded = jwt.verify(token, wpPublicKey, {
        algorithms: ['RS256']
      });
      
      // Mapear usuario de WordPress a estructura de usuario interna
      const wpUser = {
        id: decoded.sub,
        username: decoded.username || decoded.email,
        email: decoded.email,
        roles: this.mapWpRolesToInternal(decoded.roles || [])
      };
      
      return wpUser;
    } catch (error) {
      this.logger.error('Error al validar token de WordPress', { error: error.message });
      throw error;
    }
  }
  
  // Mapear roles de WordPress a roles internos
  mapWpRolesToInternal(wpRoles) {
    const roleMapping = {
      'administrator': 'admin',
      'editor': 'user',
      'author': 'user',
      'contributor': 'readonly',
      'subscriber': 'readonly'
    };
    
    return wpRoles.map(role => roleMapping[role] || 'readonly');
  }
}
```

### 6. Integración JWT de WordPress

La integración con WordPress permite autenticar usuarios desde sistemas WordPress existentes:

```javascript
// En ApiController.js - Endpoints para autenticación con WordPress

setupAuthRoutes(app) {
  app.post('/api/auth/wordpress', this.handleWordPressAuth.bind(this));
  // Otros endpoints de autenticación...
}

async handleWordPressAuth(req, res) {
  const { wpToken } = req.body;
  
  if (!wpToken) {
    return res.status(400).json({ error: 'Token de WordPress no proporcionado' });
  }
  
  try {
    const wpUser = await this.authManager.validateWordPressJWT(wpToken);
    
    // Buscar si el usuario ya existe en nuestra base de datos
    let user = await this.userRepo.findByExternalId('wordpress', wpUser.id);
    
    if (!user) {
      // Crear nuevo usuario si no existe
      user = {
        username: wpUser.username,
        email: wpUser.email,
        roles: wpUser.roles,
        externalAuth: {
          type: 'wordpress',
          id: wpUser.id
        },
        createdAt: new Date()
      };
      
      await this.userRepo.create(user);
    } else {
      // Actualizar roles si es necesario
      if (!arraysEqual(user.roles, wpUser.roles)) {
        user.roles = wpUser.roles;
        await this.userRepo.update(user.id, { roles: user.roles });
      }
    }
    
    // Generar token JWT interno
    const token = await this.authManager.createToken(user);
    
    res.json({ token, user: { id: user.id, username: user.username, roles: user.roles } });
  } catch (error) {
    this.logger.error('Error en autenticación de WordPress', { error: error.message });
    res.status(401).json({ error: 'Autenticación de WordPress fallida' });
  }
}
```

## Aspectos poco claros o pendientes

1. **Configuración de seguridad**: Se requiere definir con mayor detalle la configuración de seguridad incluyendo la generación y rotación de claves JWT.

2. **Almacenamiento de medios**: No se ha definido completamente cómo se manejarán los archivos multimedia enviados o recibidos a través de WhatsApp.

3. **Límites de rate**: Es necesario establecer límites de velocidad para evitar bloqueos por parte de WhatsApp por uso excesivo.

4. **Plan de escalabilidad**: Para manejar gran número de clientes y sesiones simultáneas, se deberá desarrollar un plan de escalabilidad horizontal.

5. **Políticas de respaldo**: Se deberán establecer políticas claras de respaldo para los datos de sesiones y mensajes.

## Próximos pasos

1. Implementar el middleware JWT y el sistema de control de acceso.
2. Desarrollar el mecanismo de recuperación de sesiones.
3. Implementar el sistema de registro por cliente.
4. Desarrollar endpoints de API para monitoreo de sesiones.
5. Integrar la autenticación con WordPress.
6. Crear la interfaz de gestión frontend.
7. Realizar pruebas de recuperación ante fallos.
8. Documentar API completa para desarrolladores.
