# Fundamentos y Metodología

## ¿Qué es OSINT?

**OSINT** (Open Source Intelligence): recopilar información de fuentes públicas disponibles: sitios web, redes sociales, registros públicos, motores de búsqueda, bases de datos, DNS, servidores WHOIS, etc.

**Reconocimiento:** fase inicial de cualquier pentesting — mapear el target sin interacción directa con sus sistemas. Objetivo: entender infraestructura, tecnologías, personas, ubicaciones, ataques potenciales.

**Diferencia Passive vs Active:**

- **Passive:** sin contacto directo con el target, no deja rastros (Google Dorking, WHOIS, Shodan, redes sociales)
- **Active:** interacción directa, el target puede detectarnos (nmap, DNS queries, scans web)

---

## Reconocimiento Pasivo

### WHOIS

Información del registrante del dominio, servidores DNS, fechas de creación/expiración.

```bash
whois google.com
whois -h whois.arin.net 8.8.8.8      # WHOIS de IP (ARIN para Norteamérica)

# Online: whois.domaintools.com
```

**Qué buscar:** registrante (nombre, email, teléfono — a veces reutilizado en otros dominios), servidores DNS (indica hosting/provider), fechas (antigüedad), registrar (GoDaddy, Namecheap, etc.).

---

### DNS Enumeration

```bash
nslookup google.com
nslookup google.com 8.8.8.8                # servidor DNS específico

dig google.com
dig google.com +short
dig google.com MX / dig google.com TXT      # registros específicos
dig @ns1.google.com google.com              # consultar un NS específico
dig google.com +trace                       # traza la resolución

host google.com
host -a google.com                          # todos los registros
host -t MX google.com
```

**Tipos de registros importantes:**

|Registro|Información|
|---|---|
|A|IPv4 del dominio|
|AAAA|IPv6 del dominio|
|MX|Servidores de correo|
|NS|Servidores DNS autoriarios|
|TXT|Texto arbitrario (SPF, DKIM, verificaciones)|
|CNAME|Alias|
|SRV|Servicios específicos|
|PTR|Reverse DNS (IP → nombre)|

**Herramientas avanzadas:**

```bash
# Sublist3r — enumera subdominios
pip install sublist3r
sublist3r -d google.com -o subdomios.txt

# DNSrecon
dnsrecon -d google.com -t std

# Fierce
fierce -d google.com

# theHarvester — recopila emails y subdominios
theHarvester -d google.com -l 500 -b google
```

---

### Google Dorking (Google Fu)

Usa operadores especiales en Google para encontrar información específica en sitios indexados.

|Operador|Uso|
|---|---|
|`site:`|Buscar solo en un dominio|
|`filetype:`|Buscar archivos de un tipo|
|`intitle:`|Buscar en títulos de página|
|`inurl:`|Buscar en URLs|
|`intext:`|Buscar en contenido de página|
|`"exact phrase"`|Frase exacta|
|`-`|Excluir término|
|`*`|Wildcard|

**Ejemplos:**

```
site:google.com                          # cualquier página de google.com
site:google.com "admin panel"
site:google.com filetype:pdf
site:google.com password / site:google.com credentials
intitle:"index of" "passwords.txt"
intitle:"Apache" "index of" site:*.com
inurl:wp-admin site:*.com                # paneles WordPress públicos
inurl:admin.php site:*.com
inurl:login filetype:php
"confidential" filetype:pdf site:*.gov
```

**Riesgos:** es legal pero ético — Google puede indexar información sensible dejada por error.

---

### Redes Sociales y Email Enumeration

```bash
# theHarvester (múltiples fuentes)
theHarvester -d example.com -l 500 -b google,bing,linkedin

# Búsqueda manual
# LinkedIn: buscar empleados, tecnologías, cargos
# Twitter/X: tweetsearch.com, tweets del dominio
# Facebook: información pública de empresas/personas
# Instagram: geolocalización, empleados, fotos de oficinas

# Email finder — encuentra emails del dominio
# email-format.com, clearbit, hunter.io (servicios online)
```

**Checklist:** Empleados (nombres, cargos, emails), tecnologías usadas (menciones en perfiles), ubicaciones, relaciones entre personas.

---

### Shodan (Motor de búsqueda de dispositivos)

Indexa dispositivos conectados a Internet: servidores, cámaras, routers, IoT, sin filtrar como Google.

```bash
# Online: shodan.io (requiere cuenta)

# CLI
shodan search "apache" --limit 10
shodan search "mongodb" --limit 5
shodan host 8.8.8.8

# Filtros útiles
shodan search "city:Madrid"
shodan search "org:Google"
shodan search "net:192.168.1.0/24"
shodan search "hostname:example.com"
shodan search "port:22"
shodan search "product:Apache"
```

**Búsquedas típicas para reconocimiento:**

- `hostname:example.com` — infraestructura del dominio
- `org:NombrEmpresa` — todos los IPs asociados
- `product:Apache country:ES` — servidores Apache en España
- `"default password"` — dispositivos con credenciales por defecto

---

### Herramientas de OSINT Integradas

```bash
# Maltego (GUI, análisis de conexiones)
# - Importar dominio/email/nombre
# - Transformaciones: DNS, whois, pasivas, activas

# SpiderFoot (CLI/GUI, recopilación automatizada)
python spiderfoot.py -l 127.0.0.1:5001

# Recon-ng (CLI, módulos, base de datos)
recon-ng
> workspace create example
> db query
```

---

## Reconocimiento Activo

### Nmap básico (ya cubierto en Redes Parte 3)

```bash
sudo nmap -sn 192.168.1.0/24           # ping sweep
nmap -p- 192.168.1.100                 # todos los puertos
nmap -sV -sC -A 192.168.1.100          # versiones, scripts, OS
```

---

### Web Server Enumeration

```bash
# Directorios y archivos
dirsearch -u http://example.com -e php,txt,html
gobuster dir -u http://example.com -w /usr/share/wordlists/dirb/common.txt
ffuf -u http://example.com/FUZZ -w wordlist.txt

# Detección de tecnologías
whatweb -a 3 http://example.com        # analiza JavaScript, headers, cookies
wappalyzer (extensión del navegador)

# Certificados SSL
echo | openssl s_client -servername example.com -connect example.com:443 | openssl x509 -noout -text
# o: crt.sh (búsqueda online de certificados)

# Headers HTTP
curl -I http://example.com
curl -v http://example.com
```

**Qué buscar:** versión de servidor web, lenguaje (PHP, ASP.NET, Node.js), frameworks, plugins, headers personalizados, cookies, comentarios en HTML.

---

### Enumeración de Usuarios

```bash
# En servicios web (tiempos de respuesta, mensajes de error)
# Fuerza bruta sobre /admin, /login, /user
hydra -L users.txt -P pass.txt http-post-form://example.com/login:username=^USER^&password=^PASS^:F=Incorrect

# Servicios de terceros (GitHub, NPM, etc.)
# Buscar proyectos/contribuciones del personal de la empresa
```

---

### Metadata de Archivos

```bash
# Descargar documentos del dominio (Google Dorking)
site:example.com filetype:pdf

# Extraer metadata
exiftool documento.pdf
exiftool -a -G1 imagen.jpg

# Buscar automáticamente
metagoofil -d example.com -t pdf,doc,xls -l 100 -n
```

**Información en metadata:** autor, software usado, fechas de creación, comentarios, ubicaciones GPS (fotos).

---

## Fases de Reconocimiento (Orden recomendado)

```
1. WHOIS — registrante, servidores DNS, fechas
2. DNS enumeration — subdominios, registros, infraestructura
3. Google Dorking — archivos indexados, información sensible
4. Redes sociales — empleados, tecnologías, ubicaciones
5. Shodan — dispositivos expuestos del dominio
6. Web server enum — directorios, versiones, tecnologías
7. Metadata — información en archivos públicos
8. Certificados SSL — nombres alternativos, historial
9. Nmap activo — puertos abiertos (si autorizado)
```

---

## Herramientas Recomendadas por Categoría

|Categoría|Herramientas|
|---|---|
|DNS/Subdomios|dig, host, nslookup, Sublist3r, DNSrecon, Fierce|
|Búsqueda web|Google Dorking, Bing, DuckDuckGo|
|Redes sociales|LinkedIn, Twitter/X, Facebook, Instagram, email-format.com|
|Dispositivos/IPs|Shodan, Censys, ZoomEye, Greynoise|
|Tecnologías|whatweb, Wappalyzer, BuiltWith|
|Directorios web|dirsearch, Gobuster, ffuf|
|Metadata|exiftool, metagoofil|
|Integradas|Maltego, Recon-ng, SpiderFoot, FOCA|
|Marcos|OSINT Framework (checklist), Awesome OSINT (GitHub)|

---

## Checklist de Reconocimiento

```
□ Información del dominio (WHOIS, DNS)
□ Subdominios y servidores asociados
□ Tecnologías (servidores, lenguajes, frameworks, plugins)
□ Información de empleados (nombres, emails, cargos, fotos)
□ Ubicaciones (oficinas, datos geográficos)
□ Servicios expuestos (Shodan, Censys)
□ Directorios y archivos públicos
□ Metadata de documentos públicos
□ Certificados SSL (nombres alternativos)
□ Historial (Wayback Machine, DNSHistory)
□ Menciones en redes sociales / forums / blogs
□ Información financiera (empresas públicas)
□ Breaches conocidos (haveibeenpwned, LeakedSource)
```

---

## Labs de Reconocimiento

### Recon sobre un dominio (ejemplo: inlanefreight.com)

```bash
# WHOIS
whois inlanefreight.com

# DNS
dig inlanefreight.com @8.8.8.8
dig inlanefreight.com MX

# Subdomios
sublist3r -d inlanefreight.com

# Google Dorking
site:inlanefreight.com filetype:pdf
site:inlanefreight.com "admin"
inurl:inlanefreight.com password

# Shodan
shodan search "hostname:inlanefreight.com"

# Tecnologías
whatweb -a 3 http://inlanefreight.com

# Directorios
gobuster dir -u http://inlanefreight.com -w /usr/share/wordlists/dirb/common.txt

# Metadata
metagoofil -d inlanefreight.com -t pdf -l 50 -n
```

Documentar: subdominios encontrados, IPs, tecnologías, empleados, directorios interesantes, archivos sensibles.

### Enumeración de empleados (LinkedIn + email finder)

```
1. LinkedIn search: "inlanefreight.com"
2. Notar: cargos, departamentos, ubicaciones
3. Hunter.io o email-format.com: firstname@inlanefreight.com
4. Verificar con smtp-user-enum, o reutilizar en ataques de phishing/social engineering
```