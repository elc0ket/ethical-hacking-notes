# Fundamentos de API Security

## ¿Qué es una API?

```
API = Application Programming Interface
Interfaz que permite comunicación entre aplicaciones

TIPOS:
• REST (Representational State Transfer)
• GraphQL (Query Language)
• SOAP (Simple Object Access Protocol)
• gRPC (Google Remote Procedure Call)
• WebSocket

USOS:
• Mobile apps ↔ Backend
• Microservicios
• Integraciones B2B
• SaaS platforms
```

---

## OWASP API Security Top 10 (2023)

```
API1:2023 - BROKEN OBJECT LEVEL AUTHORIZATION (BOLA)
• IDOR en APIs
• Acceso a objetos de otros usuarios
• Ejemplo: /api/users/123 → cambiar a 124

API2:2023 - BROKEN AUTHENTICATION
• JWT débil
• Falta de rate limiting en login
• Session management inseguro

API3:2023 - BROKEN OBJECT PROPERTY LEVEL AUTHORIZATION
• Mass assignment
• Modificar campos no autorizados
• Ejemplo: {"role": "admin"}

API4:2023 - UNRESTRICTED RESOURCE CONSUMPTION
• Sin rate limiting
• DoS posible
• Consumo excesivo de recursos

API5:2023 - BROKEN FUNCTION LEVEL AUTHORIZATION
• Acceso a funciones admin sin permisos
• Privilege escalation

API6:2023 - UNRESTRICTED ACCESS TO SENSITIVE BUSINESS FLOWS
• Automatización de flujos sensibles
• Scraping, ticket buying bots

API7:2023 - SERVER SIDE REQUEST FORGERY (SSRF)
• API hace requests controlados por atacante
• Acceso a recursos internos

API8:2023 - SECURITY MISCONFIGURATION
• CORS mal configurado
• Headers de seguridad ausentes
• Debug mode habilitado

API9:2023 - IMPROPER INVENTORY MANAGEMENT
• APIs no documentadas
• Versiones antiguas activas
• Endpoints olvidados

API10:2023 - UNSAFE CONSUMPTION OF APIs
• Confiar ciegamente en APIs externas
• Validación insuficiente
```

---

## REST API Basics

### **Métodos HTTP:**

```
GET     - Obtener recursos (idempotente)
POST    - Crear recursos
PUT     - Actualizar completo (idempotente)
PATCH   - Actualizar parcial
DELETE  - Eliminar recurso
OPTIONS - Ver métodos permitidos
HEAD    - Como GET pero sin body
```

---

### **Status Codes:**

```
2xx - SUCCESS
200 OK
201 Created
204 No Content

3xx - REDIRECTION
301 Moved Permanently
302 Found
304 Not Modified

4xx - CLIENT ERROR
400 Bad Request
401 Unauthorized (no autenticado)
403 Forbidden (no autorizado)
404 Not Found
405 Method Not Allowed
429 Too Many Requests

5xx - SERVER ERROR
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

---

### **REST API Example:**

```http
GET /api/v1/users/123 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Accept: application/json

Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "username": "john",
  "email": "john@example.com",
  "role": "user"
}
```

---

# REST API Pentesting

## Reconnaissance

### **Descubrir Endpoints:**

```bash
# Wordlists comunes
/api/
/api/v1/
/api/v2/
/rest/
/graphql
/swagger
/api-docs
/openapi.json

# Herramientas
# Dirb
dirb https://api.example.com /usr/share/wordlists/dirb/common.txt

# Gobuster
gobuster dir -u https://api.example.com/api -w api-endpoints.txt

# ffuf (rápido)
ffuf -u https://api.example.com/api/FUZZ -w api-wordlist.txt -mc 200,201,401,403

# Arjun (parameter discovery)
arjun -u https://api.example.com/api/user
```

---

### **Swagger/OpenAPI:**

```bash
# Buscar documentación
https://api.example.com/swagger
https://api.example.com/swagger.json
https://api.example.com/api-docs
https://api.example.com/v2/api-docs

# Si encuentras swagger.json → ¡GOLD!
# Todos los endpoints, parámetros, métodos

# Visualizar
https://editor.swagger.io
# Pegar el swagger.json
```

---

## BOLA/IDOR (Broken Object Level Authorization)

### **Concepto:**

```
Acceder a recursos de otros usuarios cambiando IDs

Ejemplo vulnerable:
GET /api/users/123/profile
GET /api/orders/456

Ataque:
GET /api/users/124/profile  ← Usuario diferente
GET /api/orders/457         ← Orden de otro
```

---

### **Testing BOLA:**

```bash
# 1. Crear dos cuentas de test
# User A: ID 123
# User B: ID 456

# 2. Autenticarse como User A
# Obtener token

# 3. Probar acceso a recursos de User B
curl -X GET https://api.example.com/api/users/456/profile \
  -H "Authorization: Bearer [TOKEN_USER_A]"

# Si devuelve datos de User B → BOLA vulnerable

# 4. Automatizar con Burp Intruder
# Payload: numbers 1-1000
# Buscar responses 200 OK
```

---

### **Burp Suite - BOLA Testing:**

```
1. Interceptar request
GET /api/users/123/profile HTTP/1.1
Host: api.example.com
Authorization: Bearer abc123

2. Send to Intruder

3. Mark ID position
GET /api/users/§123§/profile

4. Payloads
   Type: Numbers
   From: 1
   To: 10000
   Step: 1

5. Start Attack

6. Filtrar por:
   • Status 200
   • Length diferente (datos sensibles)
```

---

## Broken Authentication

### **JWT (JSON Web Tokens):**

```
ESTRUCTURA:
header.payload.signature

EJEMPLO:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

DECODEAR:
https://jwt.io
```

---

### **JWT Attacks:**

```bash
# 1. None Algorithm
# Cambiar "alg": "HS256" → "alg": "none"
# Eliminar signature

# Original
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJ1c2VyIn0.signature

# Modificado
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJhZG1pbiJ9.

# 2. Weak Secret (bruteforce)
# jwt-cracker
jwt-cracker "eyJhbGc..." "abcdefghijklmnopqrstuvwxyz" 6

# hashcat
hashcat -a 0 -m 16500 jwt.txt wordlist.txt

# 3. Key Confusion (RS256 → HS256)
# Si tienes public key, usarla como secret HMAC

# 4. JWT Claim Manipulation
# Cambiar: "role": "user" → "role": "admin"
# Si no verifica signature → acceso admin
```

---

### **Mass Assignment:**

```json
// Request normal
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "username": "hacker",
  "email": "hacker@example.com"
}

// Ataque: agregar campos adicionales
{
  "username": "hacker",
  "email": "hacker@example.com",
  "role": "admin",
  "is_verified": true,
  "credits": 999999
}

// Si backend no valida → privilegios admin
```

---

## Rate Limiting Bypass

### **Técnicas:**

```bash
# 1. Cambiar headers
X-Forwarded-For: 1.2.3.4
X-Real-IP: 5.6.7.8
X-Originating-IP: 9.10.11.12

# Rotar IPs en cada request
for i in {1..100}; do
  curl -H "X-Forwarded-For: 192.168.1.$i" https://api.example.com/login
done

# 2. Cambiar User-Agent
curl -H "User-Agent: Mozilla/5.0 (iPhone..." 

# 3. Null bytes en parámetros
username=admin%00&password=pass

# 4. Race condition
# Enviar múltiples requests simultáneos
# Turbo Intruder (Burp extension)

# 5. GraphQL (batch requests)
# Ver sección GraphQL
```

---

## Privilege Escalation

### **Horizontal:**

```
Acceder a recursos de usuarios del mismo nivel

User A → User B (ambos 'user' role)
```

---

### **Vertical:**

```
Elevar privilegios: user → admin

TÉCNICAS:
• Mass assignment (role: admin)
• Parameter pollution (?role=user&role=admin)
• IDOR admin endpoints (/api/admin/users)
• JWT manipulation
```