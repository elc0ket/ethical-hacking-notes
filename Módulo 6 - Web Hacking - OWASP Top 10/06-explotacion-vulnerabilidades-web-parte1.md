# Fundamentos de Aplicaciones Web

## Arquitectura Web Básica

```
Cliente (Browser)
      ↓
   [HTTP Request]
      ↓
Servidor Web (Apache, Nginx)
      ↓
Backend (PHP, Python, Node.js)
      ↓
Base de Datos (MySQL, PostgreSQL)
      ↓
   [HTTP Response]
      ↓
Cliente (Browser)
```

---

## Protocolo HTTP

### **Métodos HTTP**

```
GET     - Solicitar recurso (idempotente)
POST    - Enviar datos al servidor
PUT     - Actualizar recurso completo
PATCH   - Actualizar recurso parcialmente
DELETE  - Eliminar recurso
HEAD    - Como GET pero solo headers
OPTIONS - Métodos permitidos en el servidor
```

---

### **Request HTTP**

```http
GET /login.php HTTP/1.1
Host: ejemplo.com
User-Agent: Mozilla/5.0
Accept: text/html
Cookie: session=abc123
```

**Componentes:**

```
• Método: GET
• Path: /login.php
• Versión: HTTP/1.1
• Headers: Host, User-Agent, Cookie, etc.
• Body: (en POST/PUT)
```

---

### **Response HTTP**

```http
HTTP/1.1 200 OK
Server: Apache/2.4.41
Content-Type: text/html; charset=UTF-8
Set-Cookie: session=abc123; HttpOnly
Content-Length: 1234

<html>
...contenido...
</html>
```

**Códigos de Estado:**

```
1xx - Informacional
2xx - Éxito
    200 OK
    201 Created
3xx - Redirección
    301 Moved Permanently
    302 Found (redirect temporal)
4xx - Error del cliente
    400 Bad Request
    401 Unauthorized
    403 Forbidden
    404 Not Found
5xx - Error del servidor
    500 Internal Server Error
    502 Bad Gateway
    503 Service Unavailable
```

---

## Cookies y Sesiones

### **Cookies**

```
Pequeños datos guardados en el navegador

Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure; SameSite=Strict

Atributos:
• HttpOnly  - No accesible desde JavaScript (previene XSS)
• Secure    - Solo enviada por HTTPS
• SameSite  - Protección contra CSRF
• Domain    - Dominio válido
• Path      - Ruta válida
• Expires   - Fecha de expiración
```

---

### **Sesiones**

```
Mecanismo para mantener estado entre requests

Flujo:
1. Usuario hace login
2. Servidor crea sesión y genera ID
3. Servidor envía ID al cliente (cookie)
4. Cliente envía ID en cada request
5. Servidor valida ID y recupera datos de sesión

Vulnerabilidades:
• Session Fixation
• Session Hijacking
• Falta de expiración
• IDs predecibles
```

---

## Autenticación vs Autorización

### **Autenticación**

```
¿Quién eres?

Métodos:
• Usuario/Password
• Multi-Factor Authentication (MFA)
• Biométricos
• Certificados
• OAuth, SAML
```

### **Autorización**

```
¿Qué puedes hacer?

Tipos:
• Role-Based Access Control (RBAC)
• Attribute-Based Access Control (ABAC)
• Access Control Lists (ACL)

Ejemplo:
Usuario: john (AUTENTICADO)
Rol: admin (AUTORIZADO para ver /admin)
```

---

# OWASP Top 10

## ¿Qué es OWASP Top 10?

```
OWASP = Open Web Application Security Project

Top 10 = Las 10 vulnerabilidades más críticas en aplicaciones web

Actualizado cada 3-4 años
Última versión: 2021
Próxima: 2024 (en desarrollo)
```

**URL:** [https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/)

---

## OWASP Top 10 (2021)

```
A01:2021 - Broken Access Control
A02:2021 - Cryptographic Failures
A03:2021 - Injection
A04:2021 - Insecure Design
A05:2021 - Security Misconfiguration
A06:2021 - Vulnerable and Outdated Components
A07:2021 - Identification and Authentication Failures
A08:2021 - Software and Data Integrity Failures
A09:2021 - Security Logging and Monitoring Failures
A10:2021 - Server-Side Request Forgery (SSRF)
```

---

## A01 - Broken Access Control

### **¿Qué es?**

```
Usuarios acceden a recursos sin autorización

Ejemplos:
• Acceder a /admin sin ser admin
• Ver datos de otro usuario cambiando ID
• Modificar roles en requests
• Path traversal (../)
```

---

### **Ejemplo: IDOR (Insecure Direct Object Reference)**

```
Aplicación de banco:

GET /account?id=123
Respuesta: Datos de cuenta 123 (mi cuenta)

Ataque:
GET /account?id=124
Respuesta: Datos de cuenta 124 (cuenta de otro usuario)

¡Broken Access Control!
```

---

### **Ejemplo: Forced Browsing**

```
Usuario normal intenta acceder:

GET /admin/users
Esperado: 403 Forbidden

Si responde 200 OK y muestra contenido:
¡Broken Access Control!
```

---

### **Prevención:**

```
• Denegar por defecto
• Implementar control de acceso en backend
• Validar permisos en cada request
• No confiar en IDs del cliente
• Usar referencias indirectas
• Auditar logs de acceso
```

---

## A02 - Cryptographic Failures

### **¿Qué es?**

```
Fallas relacionadas con criptografía

Ejemplos:
• No usar HTTPS (datos en texto plano)
• Passwords sin hash
• Algoritmos débiles (MD5, SHA1)
• Claves hardcodeadas
• Datos sensibles sin cifrar en DB
```

---

### **Ejemplo: Passwords en Texto Plano**

```sql
-- MAL
CREATE TABLE users (
    id INT,
    username VARCHAR(50),
    password VARCHAR(50)  -- password123
);

-- BIEN
CREATE TABLE users (
    id INT,
    username VARCHAR(50),
    password_hash VARCHAR(255)  -- $2y$10$abc...
);
```

---

### **Prevención:**

```
• Usar HTTPS siempre
• Hash passwords con bcrypt/Argon2
• No usar MD5/SHA1 para passwords
• Cifrar datos sensibles en DB
• No hardcodear claves
• Usar TLS 1.2+
• Perfect Forward Secrecy
```

---

## A03 - Injection

### **¿Qué es?**

```
Inyectar código malicioso en la aplicación

Tipos:
• SQL Injection
• NoSQL Injection
• OS Command Injection
• LDAP Injection
• XML Injection
```

---

### **Ejemplo: SQL Injection Básico**

```php
// Código vulnerable
$username = $_POST['username'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
$result = mysqli_query($conn, $query);

// Ataque
username: admin' OR '1'='1' --
password: cualquiera

// Query resultante:
SELECT * FROM users WHERE username='admin' OR '1'='1' -- AND password='cualquiera'
// El -- comenta el resto, '1'='1' siempre es true
// Resultado: Bypass de autenticación
```

---

### **Prevención:**

```php
// Usar Prepared Statements
$stmt = $conn->prepare("SELECT * FROM users WHERE username=? AND password=?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();

// O usar ORM (Eloquent, SQLAlchemy, etc.)
User::where('username', $username)->first();
```

---

## A04 - Insecure Design

### **¿Qué es?**

```
Fallas de diseño en la arquitectura de la aplicación

Ejemplos:
• No considerar seguridad desde el inicio
• Falta de modelado de amenazas
• Límites de seguridad incorrectos
• Sin rate limiting
• Sin validación de lógica de negocio
```

---

### **Ejemplo: Lógica de Negocio**

```
E-commerce permite cupones de descuento:

Request 1:
POST /apply_coupon
{
    "code": "SAVE20",
    "order_id": 123
}
Response: 20% descuento aplicado

Request 2 (repetir):
POST /apply_coupon
{
    "code": "SAVE20",
    "order_id": 123
}
Response: 20% descuento aplicado OTRA VEZ

Total: 40% descuento (debería ser máximo 20%)
¡Falla de diseño!
```

---

## A05 - Security Misconfiguration

### **¿Qué es?**

```
Configuraciones de seguridad incorrectas o por defecto

Ejemplos:
• Credenciales por defecto (admin/admin)
• Directory listing habilitado
• Mensajes de error detallados
• Servicios innecesarios activos
• Headers de seguridad faltantes
• Software sin parches
```

---

### **Ejemplo: Directory Listing**

```
http://ejemplo.com/uploads/

Index of /uploads
  Parent Directory
  secret_document.pdf
  database_backup.sql
  passwords.txt

¡Información sensible expuesta!
```

---

### **Headers de Seguridad**

```http
# Buenos headers
Strict-Transport-Security: max-age=31536000
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'self'
X-XSS-Protection: 1; mode=block
Referrer-Policy: no-referrer
```

---

## A06 - Vulnerable and Outdated Components

### **¿Qué es?**

```
Usar librerías/frameworks con vulnerabilidades conocidas

Ejemplos:
• WordPress plugins desactualizados
• jQuery antiguo
• Log4Shell (Log4j vulnerability)
• Heartbleed (OpenSSL)
• Frameworks sin parches
```

---

### **Prevención:**

```bash
# Verificar vulnerabilidades en dependencias

# npm (Node.js)
npm audit

# pip (Python)
pip-audit

# composer (PHP)
composer audit

# Herramientas
- Snyk
- Dependabot (GitHub)
- OWASP Dependency-Check
```

---

## A07 - Identification and Authentication Failures

### **¿Qué es?**

```
Fallas en autenticación y gestión de sesiones

Ejemplos:
• Permitir contraseñas débiles
• Credenciales en URLs
• No implementar MFA
• Session fixation
• No invalidar sesiones al logout
```

---

### **Ejemplo: Sesión en URL**

```
http://ejemplo.com/dashboard?sessionid=abc123

Problemas:
• Aparece en logs del servidor
• Aparece en historial del navegador
• Aparece en referer
• Puede ser compartida

Correcto: Usar cookies HttpOnly
```

---

## A08 - Software and Data Integrity Failures

### **¿Qué es?**

```
Falta de verificación de integridad de código y datos

Ejemplos:
• Actualizar software sin verificar firma
• Usar CDN sin SRI (Subresource Integrity)
• Deserialización insegura
• CI/CD sin validación
```

---

### **Ejemplo: CDN sin SRI**

```html
<!-- Vulnerable -->
<script src="https://cdn.example.com/jquery.js"></script>

<!-- Si CDN es comprometido, código malicioso se ejecuta -->

<!-- Seguro con SRI -->
<script 
    src="https://cdn.example.com/jquery.js"
    integrity="sha384-ABC123..."
    crossorigin="anonymous">
</script>
```

---

## A09 - Security Logging and Monitoring Failures

### **¿Qué es?**

```
Falta de logs y monitoreo de seguridad

Ejemplos:
• No logear intentos de login fallidos
• No detectar ataques en curso
• Logs sin protección
• Sin alertas de seguridad
• Logs sin retención adecuada
```

---

## A10 - Server-Side Request Forgery (SSRF)

### **¿Qué es?**

```
Hacer que el servidor realice requests a recursos internos

Flujo:
1. Aplicación permite especificar URL
2. Servidor hace request a esa URL
3. Atacante especifica URL interna
4. Servidor accede a recursos internos
```

---

### **Ejemplo: SSRF Básico**

```php
// Código vulnerable
$url = $_GET['url'];
$content = file_get_contents($url);
echo $content;

// Ataque
?url=http://localhost/admin
?url=http://169.254.169.254/latest/meta-data/  (AWS metadata)
?url=file:///etc/passwd

// Servidor accede a recursos que el atacante no podría
```

---

### **Prevención de SSRF:**

```
• Whitelist de URLs permitidas
• No permitir IPs privadas (127.0.0.1, 192.168.x.x)
• Validar protocolos (solo http/https)
• Usar biblioteca de validación de URLs
• Network segmentation
```