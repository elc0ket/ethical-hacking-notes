# OSINT de Personas y Metadatos

## Recolección de Emails

### theHarvester

Recopila emails, subdominios y hosts de múltiples fuentes públicas.

```bash
# Instalar
sudo apt install theharvester

# Buscar emails de un dominio
theHarvester -d ejemplo.com -b all                  # todas las fuentes
theHarvester -d ejemplo.com -b google -l 500        # solo Google, máximo 500 resultados
theHarvester -d ejemplo.com -b bing,google,linkedin
theHarvester -d ejemplo.com -b all -f resultado.txt # guardar a archivo
```

|Opción|Descripción|
|---|---|
|`-d`|Dominio|
|`-b`|Fuente (google, bing, linkedin, github, twitter, shodan, etc.)|
|`-l`|Límite de resultados|
|`-f`|Guardar a archivo|
|`-p`|Usar proxy|

**Fuentes disponibles:** google, bing, baidu, yandex, linkedin, github, shodan, hunter, otx, security.trade, etc.

---

### Hunter.io

Web: https://hunter.io — identifica patrones de emails de un dominio.

|Método|Descripción|
|---|---|
|**Domain Search**|Encuenta todos los emails del dominio|
|**Email Finder**|Busca email de una persona específica|
|**Email Verifier**|Verifica si un email es válido|
|**API**|Integración programática|

**Ventajas:** alta precisión, historial de búsqueda, exportación en CSV.

---

### Email Pattern Guessing

Si el dominio usa patrón `firstname.lastname@example.com`, se pueden generar variaciones y verificar con SMTP:

```bash
# Generar lista de posibles emails
# Combinaciones: firstname.lastname, f.lastname, firstname_lastname, flastname, etc.

# Verificar SMTP (sin enviar)
sudo apt install smtp-user-enum
smtp-user-enum -M VRFY -U usuarios.txt -t ejemplo.com

# O con Python
for email in emails.txt; do
    timeout 2 bash -c "echo -e 'EHLO test\nMAIL FROM: test@test.com\nRCPT TO: $email' | nc ejemplo.com 25" 2>/dev/null | grep -q "250" && echo "[+] $email válido"
done
```

---

### LinkedIn Intelligence

**Búsqueda manual en LinkedIn:**

```
1. Company search → empleados
2. Skills → tecnologías usadas
3. Cambios de rol → puntos débiles (empleados nuevos, poca experiencia)
4. Foto de perfil → reverse image search (otros perfiles, ubicaciones)
5. Ubicación → domicilio aproximado
```

**Herramientas (LinkedIn scraping — verificar T&C):**

```bash
# linkedin2username
linkedin2username -c "cookies.txt" -u empresa_url > empleados.txt

# lixi
# Nota: LinkedIn bloquea scraping automatizado — métodos activos suelen fallar
```

---

### Google Dorking para Emails

```
# Buscar emails expuestos públicamente
site:pastebin.com "example.com"
site:github.com "example.com" email
intext:"@example.com"
intitle:"admin" "example.com"
```

---

# Shodan — Motor de Búsqueda de Dispositivos

Indexa dispositivos conectados a Internet sin filtros: servidores, cámaras, routers, impresoras, sistemas SCADA, etc.

### Filtros Útiles

|Filtro|Ejemplo|
|---|---|
|`hostname:`|`hostname:ejemplo.com`|
|`port:`|`port:3389`|
|`country:`|`country:CO`|
|`city:`|`city:"Madrid"`|
|`product:`|`product:Apache`|
|`org:`|`org:"Nombre Empresa"`|
|`http.title:`|`http.title:"admin panel"`|
|`http.favicon.hash:`|Para aplicaciones específicas|
|`ssl.cert.subject.CN:`|Certificados SSL|

### Ejemplos de Búsquedas

```
product:"MongoDB" port:27017              # Bases de datos expuestas
apache version:2.4.49                     # Servidores Apache específicos
http.title:"admin panel"                  # Paneles de administración
"mikrotik" country:CO                     # Routers MikroTik en Colombia
"cisco" port:443                          # Dispositivos Cisco
hostname:ejemplo.com                       # Infraestructura del dominio
```

### Shodan CLI

```bash
# Instalar
pip3 install shodan

# Configurar API
shodan init API_KEY

# Búsquedas
shodan search "apache country:CO" --limit 10
shodan search "mongodb port:27017"
shodan host 8.8.8.8                       # Info de una IP específica
shodan domain ejemplo.com                 # Búsqueda rápida por dominio

# Convertir resultados
shodan search "apache" --limit 100 | jq '.matches[] | .ip_str' > ips.txt
```

**Checklist Shodan para reconocimiento:**

```
□ hostname:target.com                     → IPs asociadas
□ org:"Nombre Empresa"                    → Todos los servicios de la empresa
□ product:Apache                          → Versión específica, posibles CVEs
□ http.title:"admin"                      → Paneles accesibles
□ ssl.cert.subject.CN:target.com          → Certificados y nombres alternativos
□ port:3389 country:CO                    → RDP abiertos en país específico
```

---

# Maltego — Análisis Visual de Relaciones

Herramienta GUI para mapear y analizar relaciones entre entidades de OSINT.

### Entidades principales

|Entidad|Uso|
|---|---|
|Person|Personas (nombre, email, teléfono)|
|Email|Direcciones de correo|
|Domain|Dominios|
|IP Address|Direcciones IP|
|Company|Empresas|
|Website|Sitios web|
|Phone|Números telefónicos|
|Social Network Account|Perfiles en redes|

### Transformaciones (Transforms)

|Transform|Entrada → Salida|
|---|---|
|Domain to IP|Dominio → IPs asociadas|
|Email to Person|Email → personas que lo usan|
|Person to Email|Persona → emails relacionados|
|Website to IP|Sitio web → servidor IP|
|IP to Domain|IP → dominios alojados|
|Email to Social Accounts|Email → cuentas en redes|

### Flujo de Análisis Típico

```
1. Importar entidad: ejemplo.com (dominio)
2. Run Transform: DNS → obtener MX, NS, A records
3. Dominio → IPs
4. IPs → hosts (reverse DNS)
5. Hosts → subdominios
6. Emails encontrados → personas
7. Personas → empleos, ubicaciones, redes sociales
```

---

# Recon-ng — Framework de Reconocimiento Modular

Framework de línea de comandos con módulos de reconocimiento.

```bash
# Instalar
sudo apt install recon-ng

# Iniciar
recon-ng

# Dentro de recon-ng:
```

|Comando|Función|
|---|---|
|`workspaces create nombre`|Crear workspace|
|`workspaces list`|Ver workspaces|
|`workspaces select nombre`|Cambiar workspace|
|`marketplace search tipo`|Buscar módulos|
|`marketplace install módulo`|Instalar módulo|
|`modules load módulo`|Cargar módulo|
|`options set OPTION valor`|Configurar opciones|
|`run`|Ejecutar módulo|
|`show hosts / show emails / show contacts`|Ver resultados|
|`db query`|Consultar la BD|

### Ejemplo: Reconocimiento de Dominio

```bash
recon-ng
> workspaces create ejemplo
> marketplace install recon/domains-hosts/hackertarget
> modules load recon/domains-hosts/hackertarget
> options set SOURCE ejemplo.com
> run
> show hosts
> show emails
```

### Módulos Útiles

|Módulo|Función|
|---|---|
|`recon/domains-hosts/hackertarget`|Subdominios desde HackerTarget|
|`recon/domains-hosts/googleCSE`|Subdominios desde Google CSE|
|`recon/domains-contacts/metacrawler`|Contactos de dominio|
|`recon/hosts-domains/reverse_resolve`|Reverse DNS|
|`recon/companies-contacts/linkedin`|Contactos de LinkedIn (requiere auth)|
|`reporting/html`|Generar reporte HTML|

---

# Análisis de Metadatos

### ExifTool

Extrae, edita y elimina metadatos de archivos (imágenes, documentos).

```bash
# Instalar
sudo apt install libimage-exiftool-perl

# Ver todos los metadatos
exiftool imagen.jpg
exiftool documento.pdf

# Ver metadatos específicos
exiftool -GPS* foto.jpg                   # coordenadas GPS
exiftool -a -G1 imagen.jpg                # grupo y etiqueta

# GPS en decimal
exiftool -n -GPS* foto.jpg

# Remover todos los metadatos
exiftool -all= imagen_sin_metadata.jpg

# Remover metadatos específicos
exiftool -GPS*= foto.jpg
exiftool -Author= documento.pdf

# Copiar metadatos entre archivos
exiftool -TagsFromFile original.jpg destino.jpg
```

### Información Típica en Metadatos

**Imágenes (EXIF):**

|Dato|Información|
|---|---|
|GPS|Coordenadas exactas de dónde se tomó la foto|
|DateTime|Fecha y hora exacta|
|Model|Modelo de cámara / smartphone|
|Software|Aplicación de edición usada|
|Copyright|Autor, empresa|
|Orientation|Cómo se tomó (rotación)|

**Documentos (PDF, Word):**

|Dato|Información|
|---|---|
|Author|Quien creó el documento|
|Company|Empresa del autor|
|Creator|Software que lo creó|
|CreationDate|Cuándo se creó|
|LastModifiedBy|Quién lo editó por último|
|Comments|Comentarios ocultos|
|Template|Template usado (puede indicar versión de Office)|
|Embedded Objects|Archivos/objetos dentro|

### Herramientas Adicionales

```bash
# FOCA (GUI, busca automáticamente metadatos)
sudo apt install foca

# metagoofil (busca y descarga documentos de un dominio)
metagoofil -d ejemplo.com -t pdf,doc,xls -l 50 -n
```

---

## Técnicas de Reverse Image Search

Encontrar información sobre fotos de empleados o ubicaciones:

```bash
# Google Images (manual)
https://images.google.com → cargar imagen

# TinEye
https://tineye.com

# Yandex Images
https://yandex.com/images

# Bing Images
https://bing.com/images

# CLI (requiere API o scraping)
# ris-cli (GitHub)
```

**Uso:** empleado en foto de perfil → búsqueda inversa → otros perfiles, ubicaciones, nombres reales.

---

## Checklist de OSINT de Personas

```
□ Emails (theHarvester, Hunter, Google Dorking)
□ LinkedIn (empleos, ubicación, conexiones)
□ Redes sociales (Twitter, Instagram, Facebook, TikTok)
□ Fotografías (reverse image search, metadatos GPS)
□ Números de teléfono (búsqueda inversa, truecaller)
□ Direcciones (Google Maps, registros públicos)
□ Información laboral (currículos expuestos, portafolios)
□ Historiales (Wayback Machine, archive.is)
□ Breaches (haveibeenpwned, breach databases)
□ Menciones públicas (prensa, blogs, foros)
□ Proyectos de código (GitHub, GitLab, personal repos)
□ Documentos públicos (PDFs expuestos con metadata)
```

---

## Labs de OSINT de Personas

### Recon sobre empleado ficticio: John Smith, ejemplo.com

```bash
# 1. Encontrar emails
theHarvester -d ejemplo.com -b all
hunter.io: buscar "john smith" en ejemplo.com

# 2. LinkedIn
Buscar: John Smith + ejemplo.com
Notar: ubicación, empleos anteriores, conexiones

# 3. Redes sociales
Twitter: @johnsmith + ejemplo.com
Facebook: John Smith + ejemplo.com
LinkedIn URL + reverse search

# 4. Fotos
Descargar foto de perfil de LinkedIn
Reverse image search (Google, TinEye, Yandex)

# 5. Emails exposición
site:pastebin.com "john.smith@ejemplo.com"
site:github.com "john.smith@ejemplo.com"

# 6. Breaches
haveibeenpwned.com: john.smith@ejemplo.com

# 7. Documentos públicos
site:ejemplo.com filetype:pdf "John Smith"
metagoofil -d ejemplo.com -t pdf
exiftool descargado.pdf (buscar autor, empresa, software)

# 8. Maltego
Entidad: Email (john.smith@ejemplo.com)
Transform: Email to Person
Transform: Person to Website
```

Documentar: todo lo encontrado, fuente, fecha.

---

## Herramientas Dominadas en Parte 2

```
✓ theHarvester (emails automatizadas)
✓ Hunter.io (verificación de emails)
✓ Shodan (dispositivos expuestos)
✓ Shodan CLI (búsquedas programáticas)
✓ Maltego (análisis visual de relaciones)
✓ Recon-ng (framework modular)
✓ ExifTool (análisis de metadatos)
✓ Reverse Image Search (ubicar información de fotos)
✓ Google Dorking avanzado (emails, documentos)
✓ Técnicas de Pattern Guessing (generación de emails)
``