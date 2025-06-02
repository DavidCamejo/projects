# Documentación Técnica: Plugin de Integración WhatsApp para WordPress

## 1. Introducción

El plugin "WhatsApp API for WooCommerce" permite integrar de forma sencilla las funcionalidades de mensajería de WhatsApp en sitios WordPress con WooCommerce. Esta integración facilita la comunicación automatizada con clientes, mejora la experiencia de usuario y optimiza procesos de negocio relacionados con ventas y atención al cliente.

## 2. Requisitos Previos del Sistema

### Requisitos de Software
- WordPress 5.6 o superior
- PHP 7.2 o superior
- WooCommerce 4.0 o superior
- Base de datos MySQL 5.6 o MariaDB 10.0 o superior
- Servidor web Apache o Nginx

### Requisitos de Acceso
- Acceso al panel de administración de WordPress
- Privilegios para instalar y activar plugins
- Acceso al sistema de archivos del sitio web (recomendado para solucionar problemas)

### Requisitos de API
- Una instancia en funcionamiento de WhatsApp API Manager
- URL base de la API
- Credenciales de autenticación válidas (usuario/contraseña)

## 3. Instrucciones de Instalación

### Método 1: Instalación desde el Repositorio de WordPress

1. Acceda al panel de administración de WordPress
2. Navegue a "Plugins" -> "Añadir nuevo"
3. En la barra de búsqueda, escriba "WhatsApp API for WooCommerce"
4. Haga clic en "Instalar ahora" junto al plugin
5. Una vez instalado, haga clic en "Activar"

### Método 2: Instalación Manual (Archivo ZIP)

1. Descargue la última versión del plugin desde el repositorio oficial
2. Acceda al panel de administración de WordPress
3. Navegue a "Plugins" -> "Añadir nuevo"
4. Haga clic en "Subir plugin"
5. Seleccione el archivo ZIP descargado
6. Haga clic en "Instalar ahora"
7. Una vez instalado, haga clic en "Activar"

### Método 3: Instalación mediante FTP

1. Descargue y descomprima el plugin
2. Conecte a su servidor mediante FTP
3. Navegue al directorio `/wp-content/plugins/`
4. Suba la carpeta `wp-whatsapp-integration` al directorio de plugins
5. Acceda al panel de administración de WordPress
6. Navegue a "Plugins" y active "WhatsApp API for WooCommerce"

## 4. Configuración Inicial

### Conexión con la API de WhatsApp

1. Después de activar el plugin, navegue a "WooCommerce" -> "WhatsApp API" en el menú de administración
2. En la pestaña "Configuración general", introduzca los siguientes datos:
   - URL de la API: La dirección base de su instancia de WhatsApp API Manager
   - Tiempo de espera de conexión: Tiempo máximo para conexiones API (recomendado: 30 segundos)
   - Número máximo de reintentos: Cantidad de intentos para operaciones fallidas (recomendado: 3)
3. Haga clic en "Guardar cambios"

### Generación de Secretos y Tokens

1. En la pestaña "Seguridad", haga clic en "Generar clave secreta JWT"
   - Esta clave se utilizará para firmar tokens de autenticación entre WordPress y la API
2. Haga clic en "Generar API Key" 
   - Esta clave se utiliza para identificar solicitudes de su sitio a la API
3. Copie ambas claves y guárdelas en un lugar seguro
4. Para probar la conexión, haga clic en "Verificar conexión"

### Configuración de Permisos

El plugin incluye un sistema de control de acceso basado en roles. Por defecto, los siguientes roles pueden acceder al sistema:
- Administrator
- Vendor
- Shop Manager
- Vendor Admin
- Vendor Staff
- WCFM Vendor

Puede modificar los roles permitidos utilizando el filtro de WordPress `wpwa_allowed_roles`.

## 5. Funciones Principales

### Gestión de Sesiones

#### Creación de una Sesión de WhatsApp

1. Navegue a "WooCommerce" -> "WhatsApp API" -> "Sesiones"
2. Haga clic en "Crear nueva sesión"
3. Introduzca un nombre identificativo para la sesión
4. Escanee el código QR mostrado con su aplicación de WhatsApp en el teléfono
5. Espere a que el sistema confirme la conexión

#### Gestión de Sesiones Existentes

- **Ver estado**: Compruebe si una sesión está activa, en espera o desconectada
- **Reconectar**: Restablezca una sesión desconectada
- **Eliminar**: Elimine completamente una sesión del sistema

### Envío de Mensajes

#### Mensajes de Texto Simple

Utilice la función `wpwa()->message_manager->send_text_message()` para enviar mensajes de texto:

```php
$cliente_id = 'ID_DE_SESION';
$destinatario = '34600000000@c.us'; // Número con prefijo internacional + @c.us
$mensaje = 'Hola, este es un mensaje de prueba';

$resultado = wpwa()->message_manager->send_text_message($cliente_id, $destinatario, $mensaje);
```

#### Mensajes con Archivos Adjuntos

Para enviar imágenes, documentos u otros archivos:

```php
$cliente_id = 'ID_DE_SESION';
$destinatario = '34600000000@c.us';
$url_archivo = 'https://ejemplo.com/imagen.jpg';
$caption = 'Descripción de la imagen'; // Opcional

$resultado = wpwa()->message_manager->send_media_message($cliente_id, $destinatario, $url_archivo, $caption);
```

#### Mensajes de Producto

Para enviar información de productos de WooCommerce:

```php
$cliente_id = 'ID_DE_SESION';
$destinatario = '34600000000@c.us';
$product_id = 123; // ID del producto en WooCommerce

$resultado = wpwa()->message_manager->send_product_message($cliente_id, $destinatario, $product_id);
```

### Automatización de Mensajes

#### Notificaciones de Estado de Pedido

El plugin permite enviar automáticamente mensajes cuando cambia el estado de un pedido:

1. Navegue a "WooCommerce" -> "WhatsApp API" -> "Automatización"
2. Active la opción "Notificaciones de estado de pedido"
3. Configure los estados para los que desea enviar notificaciones (Procesando, Completado, etc.)
4. Personalice las plantillas de mensaje para cada estado

### Sistema de Plantillas

Cree y gestione plantillas de mensajes reutilizables:

1. Navegue a "WooCommerce" -> "WhatsApp API" -> "Plantillas"
2. Haga clic en "Añadir nueva plantilla"
3. Complete el formulario con:
   - Nombre de la plantilla
   - Contenido del mensaje (puede incluir variables como {{customer_name}})
   - Idioma
   - Categoría
4. Guarde la plantilla

Para usar la plantilla en código:

```php
$cliente_id = 'ID_DE_SESION';
$destinatario = '34600000000@c.us';
$template_id = 'ID_PLANTILLA';
$variables = array('customer_name' => 'Juan Pérez');

$resultado = wpwa()->message_manager->send_template_message($cliente_id, $destinatario, $template_id, $variables);
```

## 6. Integración con WooCommerce

### Botones de WhatsApp en Páginas de Producto

Para añadir un botón "Preguntar por WhatsApp" en páginas de producto:

1. Navegue a "WooCommerce" -> "WhatsApp API" -> "Integración"
2. Active la opción "Mostrar botón en páginas de producto"
3. Personalice el texto y estilo del botón

También puede usar el shortcode `[wpwa_product_button]` para colocar el botón en cualquier ubicación.

### Seguimiento de Carros Abandonados

Para activar recordatorios para carritos abandonados:

1. Navegue a "WooCommerce" -> "WhatsApp API" -> "Automatización"
2. Active la opción "Recordatorios de carrito abandonado"
3. Configure el tiempo de espera y el mensaje a enviar

## 7. Solución de Problemas Comunes

### Problemas de Conexión con la API

**Síntoma**: Error "No se puede conectar con el servidor API"

**Soluciones**:
- Verifique que la URL de la API sea correcta y esté accesible
- Compruebe que el servidor API esté en funcionamiento
- Revise la configuración de firewall del servidor
- Aumente el tiempo de espera de conexión en la configuración del plugin

### Error en la Generación de Tokens JWT

**Síntoma**: Error "Error generando token JWT"

**Soluciones**:
- Regenere la clave secreta JWT en la configuración
- Verifique que el usuario tenga los roles permitidos
- Revise los logs para obtener más detalles sobre el error

### Problemas al Escanear Código QR

**Síntoma**: El código QR no se muestra o no se puede escanear correctamente

**Soluciones**:
- Refresque la página para generar un nuevo código QR
- Asegúrese de que el navegador tiene conexión a Internet
- Pruebe con otro navegador
- Verifique que la sesión no esté ya conectada

### Errores en Envío de Mensajes

**Síntoma**: Los mensajes no se envían correctamente

**Soluciones**:
- Compruebe que la sesión de WhatsApp esté activa
- Verifique que el formato del número de teléfono sea correcto (incluido prefijo internacional)
- Revise los logs del plugin para obtener detalles del error
- Asegúrese de que el mensaje no incumple las políticas de WhatsApp

## 8. Logs y Depuración

### Activar Modo de Depuración

1. Navegue a "WooCommerce" -> "WhatsApp API" -> "Configuración"
2. Active la opción "Modo de depuración"
3. Guarde los cambios

### Acceder a los Logs

Los logs se almacenan en `/wp-content/uploads/wpwa-logs/`

Para ver los logs desde el panel de administración:
1. Navegue a "WooCommerce" -> "WhatsApp API" -> "Logs"
2. Seleccione el archivo de log que desea consultar

## 9. Seguridad

### Protección de Tokens JWT

El plugin utiliza tokens JWT (JSON Web Tokens) para la autenticación segura, con las siguientes características:

- Tokens firmados con algoritmo HS256
- Codificación Base64URL segura para transmisión en URLs y headers HTTP
- Expiración configurable (por defecto: 1 hora)
- Control de acceso basado en roles
- Blacklisting de tokens revocados

### Almacenamiento Seguro de Credenciales

Las claves secretas y credenciales se almacenan cifradas en la base de datos de WordPress.

## 10. Información Adicional

### Configuración Avanzada mediante hooks

El plugin proporciona múltiples hooks y filtros para personalizar su funcionamiento:

```php
// Modificar los roles permitidos para usar la API
add_filter('wpwa_allowed_roles', function($roles) {
    $roles[] = 'custom_role';
    return $roles;
});

// Personalizar el payload del token JWT
add_filter('wpwa_jwt_payload', function($payload, $user_id) {
    $payload['custom_data'] = get_user_meta($user_id, 'custom_field', true);
    return $payload;
}, 10, 2);
```

### Actualizaciones

El plugin se actualiza regularmente para añadir nuevas funcionalidades y corregir posibles errores. Se recomienda mantenerlo siempre actualizado a la última versión disponible.

Para actualizar:
1. Haga una copia de seguridad de su sitio
2. Navegue a "Plugins" -> "Plugins instalados"
3. Si hay una actualización disponible, aparecerá un aviso junto al plugin
4. Haga clic en "Actualizar ahora"

### Compatibilidad con Plugins de Marketplace

El plugin es compatible con los siguientes plugins de marketplace para WooCommerce:

- WCFM Marketplace
- Dokan
- WC Vendors

Para cada uno de estos plugins, se integra automáticamente para permitir que los vendors gestionen sus propias sesiones de WhatsApp y se comuniquen con sus clientes.