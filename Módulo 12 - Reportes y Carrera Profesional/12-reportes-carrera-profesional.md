# Metodologías de Pentesting

## PTES (Penetration Testing Execution Standard)

### **Fases del PTES:**

```
1. PRE-ENGAGEMENT
   • Definir alcance
   • Reglas de engagement
   • Objetivos del test
   • Contactos de emergencia

2. INTELLIGENCE GATHERING
   • OSINT
   • Reconocimiento pasivo
   • Footprinting

3. THREAT MODELING
   • Identificar activos valiosos
   • Posibles vectores de ataque
   • Impacto potencial

4. VULNERABILITY ANALYSIS
   • Escaneo de vulnerabilidades
   • Validación manual
   • Priorización

5. EXPLOITATION
   • Aprovechar vulnerabilidades
   • Obtener acceso
   • Privilege escalation

6. POST-EXPLOITATION
   • Mantener acceso
   • Pivotar
   • Exfiltrar datos (simulado)

7. REPORTING
   • Documentar hallazgos
   • Recomendaciones
   • Entrega ejecutiva y técnica
```

---

## OWASP Testing Guide

### **Para Aplicaciones Web:**

```
1. INFORMATION GATHERING
   • Fingerprinting
   • Mapeo de aplicación
   • Identificar puntos de entrada

2. CONFIGURATION MANAGEMENT
   • Archivos de backup
   • Métodos HTTP
   • Headers de seguridad

3. IDENTITY MANAGEMENT
   • Enumeración de usuarios
   • Políticas de contraseñas

4. AUTHENTICATION
   • Credenciales por defecto
   • Bypass de autenticación
   • Session management

5. AUTHORIZATION
   • Path traversal
   • Privilege escalation
   • Insecure Direct Object References

6. SESSION MANAGEMENT
   • Cookies
   • Session fixation
   • CSRF

7. INPUT VALIDATION
   • SQL Injection
   • XSS
   • Command Injection
   • XXE, SSRF

8. ERROR HANDLING
   • Mensajes de error detallados
   • Stack traces

9. CRYPTOGRAPHY
   • SSL/TLS
   • Datos sensibles sin cifrar

10. BUSINESS LOGIC
    • Flujos anómalos
    • Validación inadecuada

11. CLIENT-SIDE TESTING
    • JavaScript
    • HTML5
    • Almacenamiento local
```

---

## NIST SP 800-115

### **Framework de NIST:**

```
1. PLANNING
   • Coordinación
   • Alcance definido
   • Aprobaciones

2. DISCOVERY
   • Network discovery
   • Service identification
   • Vulnerability scanning

3. ATTACK
   • Ganar acceso
   • Escalar privilegios
   • Browse y exfiltrar

4. REPORTING
   • Hallazgos técnicos
   • Recomendaciones
   • Métricas
```

---

## MITRE ATT&CK Framework

### **Tácticas y Técnicas:**

```
TACTICS (Objetivos):
1. Initial Access
2. Execution
3. Persistence
4. Privilege Escalation
5. Defense Evasion
6. Credential Access
7. Discovery
8. Lateral Movement
9. Collection
10. Command and Control
11. Exfiltration
12. Impact

EJEMPLO - Initial Access:
• T1566.001 - Spearphishing Attachment
• T1566.002 - Spearphishing Link
• T1190 - Exploit Public-Facing Application

EJEMPLO - Credential Access:
• T1003 - OS Credential Dumping
• T1110 - Brute Force
• T1558 - Kerberoasting

Mapear tus hallazgos a ATT&CK
```

---

# Reportes Profesionales

## Estructura de Reporte de Pentesting

### **1. Executive Summary**

```
Audiencia: C-level, management

Contenido:
• Objetivo del test
• Alcance
• Metodología utilizada
• Resumen de hallazgos (high-level)
• Riesgo general
• Recomendaciones principales

Longitud: 1-2 páginas máximo

Ejemplo:
"Durante el periodo del 15-20 de enero de 2026, se realizó 
una evaluación de seguridad del entorno de la aplicación 
web XYZ. Se identificaron 3 vulnerabilidades críticas, 
8 altas y 15 medianas que podrían permitir a un atacante 
comprometer datos sensibles de clientes..."
```

---

### **2. Scope**

```
• Direcciones IP evaluadas
• Dominios incluidos
• Aplicaciones específicas
• Credenciales proporcionadas (si aplica)
• Exclusiones
• Timeframe
• Tipo de test (black box, gray box, white box)

Ejemplo:
INCLUSIONS:
• www.ejemplo.com
• api.ejemplo.com
• 192.168.1.0/24

EXCLUSIONS:
• legacy.ejemplo.com (out of service)
• Pruebas de DoS
• Social engineering

TYPE: Gray Box (credenciales de usuario regular)
DATES: 15-20 Enero 2026
```

---

### **3. Methodology**

```
• Frameworks seguidos (PTES, OWASP)
• Herramientas utilizadas
• Fases del test

Ejemplo:
"Se siguió la metodología PTES, utilizando herramientas 
estándar de la industria como Nmap, Burp Suite Professional, 
SQLMap y Metasploit Framework. El test se dividió en 
las siguientes fases:
1. Reconnaissance
2. Vulnerability Assessment
3. Exploitation
4. Post-Exploitation
5. Reporting"
```

---

### **4. Findings Summary**

```
Tabla resumen:

| Severidad | Cantidad |
|-----------|----------|
| Critical  | 3        |
| High      | 8        |
| Medium    | 15       |
| Low       | 22       |
| Info      | 10       |
| **TOTAL** | **58**   |

Risk Rating Chart (visual)
```

---

### **5. Detailed Findings**

```
POR CADA VULNERABILIDAD:

TÍTULO
• Descriptivo y específico

SEVERITY
• Critical / High / Medium / Low / Informational
• CVSS Score (ej: 9.8)

DESCRIPTION
• Qué es la vulnerabilidad
• Por qué es un problema

LOCATION
• URL específica
• Parámetro vulnerable
• IP y puerto

IMPACT
• Qué puede hacer un atacante
• Datos en riesgo
• Sistemas afectados

PROOF OF CONCEPT
• Request/Response
• Screenshot
• Pasos para reproducir

REMEDIATION
• Cómo arreglarlo
• Específico y accionable

REFERENCES
• OWASP, CWE, CVE
• Links a documentación
```

---

### **Ejemplo de Finding:**

````markdown
# SQL Injection in Login Form

**Severity:** Critical (CVSS 9.8)

**Description:**
The login form at /admin/login.php is vulnerable to SQL 
Injection. An attacker can bypass authentication and gain 
unauthorized access to the admin panel by injecting SQL 
commands in the username parameter.

**Location:**
- URL: https://ejemplo.com/admin/login.php
- Parameter: username (POST)

**Impact:**
- Complete database compromise
- Unauthorized admin access
- Potential data exfiltration of customer PII
- Ability to modify/delete records

**Proof of Concept:**
```http
POST /admin/login.php HTTP/1.1
Host: ejemplo.com
Content-Type: application/x-www-form-urlencoded

username=admin' OR '1'='1' -- &password=anything
````

Response: 302 Redirect to /admin/dashboard.php

**Screenshot:** [Attached]

**Remediation:**

1. Use parameterized queries (prepared statements)
2. Implement input validation and sanitization
3. Apply principle of least privilege for DB user
4. Enable WAF rules for SQL injection

**Code Example (Secure):**

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$username]);
```

**References:**

- OWASP Top 10 2021: A03 Injection
- CWE-89: SQL Injection
- CAPEC-66: SQL Injection

---

### **6. Recommendations**

```

IMMEDIATE (Critical/High):

1. Patch SQL injection vulnerabilities
2. Disable default credentials
3. Apply security updates

SHORT-TERM (1-3 months):

1. Implement WAF
2. Security awareness training
3. MFA for admin accounts

LONG-TERM (3-6 months):

1. Security code review process
2. Regular penetration testing
3. SIEM implementation
```

---

### **7. Appendices**

```
A. Complete Scan Results B. Tool Outputs C. Risk Rating Methodology D. Testing Timeline E. Contact Information
```

---

## Formato Visual

```

BEST PRACTICES:

✅ Logo y branding profesional ✅ Tabla de contenidos ✅ Screenshots de calidad ✅ Código con syntax highlighting ✅ Tablas y gráficos claros ✅ Colores para severidad: • Critical: Rojo • High: Naranja • Medium: Amarillo • Low: Azul • Info: Gris

✅ Headers y footers con: • Número de página • CONFIDENTIAL • Nombre de empresa • Fecha

❌ EVITAR: • Errores ortográficos • Screenshots borrosos • Demasiado jerga técnica en executive summary • Información incompleta

```

---

## Herramientas para Reportes

```

REPORTE ESCRITO: • Microsoft Word • Google Docs • LaTeX (profesional)

GENERADORES: • Dradis Framework (colaborativo) • Serpico • PlexTrac • Faraday

VISUALIZACIÓN: • Matplotlib (Python) - gráficos • Chart.js - web • Excel/Google Sheets - tablas

TEMPLATES: • SANS Penetration Testing Report Template • PTES Sample Report • Custom templates corporativos

```

---

# CVSS y Severidad

## CVSS (Common Vulnerability Scoring System)

### **CVSS v3.1 - Métricas Base**

```

ATTACK VECTOR (AV): • Network (N) - 0.85 • Adjacent (A) - 0.62 • Local (L) - 0.55 • Physical (P) - 0.2

ATTACK COMPLEXITY (AC): • Low (L) - 0.77 • High (H) - 0.44

PRIVILEGES REQUIRED (PR): • None (N) - 0.85 • Low (L) - 0.62 • High (H) - 0.27

USER INTERACTION (UI): • None (N) - 0.85 • Required (R) - 0.62

SCOPE (S): • Unchanged (U) • Changed (C) - Mayor impacto

CONFIDENTIALITY (C): • None (N) - 0 • Low (L) - 0.22 • High (H) - 0.56

INTEGRITY (I): • None (N) - 0 • Low (L) - 0.22 • High (H) - 0.56

AVAILABILITY (A): • None (N) - 0 • Low (L) - 0.22 • High (H) - 0.56

```

---

### **Cálculo de Score:**

```

FORMULA (simplificada): Base Score = Impact Sub Score + Exploitability Sub Score

RANGOS: 0.0 None 0.1-3.9 Low 4.0-6.9 Medium 7.0-8.9 High 9.0-10.0 Critical

```

---

### **Ejemplos:**

```

SQL INJECTION (sin autenticación): AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H Score: 10.0 (Critical)

XSS REFLECTED: AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N Score: 6.1 (Medium)

PRIVILEGE ESCALATION (local): AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H Score: 7.8 (High)

```

---

### **Calculadora CVSS:**

```

Online: https://www.first.org/cvss/calculator/3.1

Proceso:

1. Seleccionar cada métrica
2. Score se calcula automáticamente
3. Incluir vector string en reporte