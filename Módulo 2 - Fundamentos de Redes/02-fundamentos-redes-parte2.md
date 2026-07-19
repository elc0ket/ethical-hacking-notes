# Protocolos, Subnetting y Routing

## DNS (Domain Name System)

Traduce nombres de dominio a IPs (`google.com → 142.250.65.206`) — la "agenda telefónica" de Internet.

### Proceso de resolución

```
Usuario pide "google.com"
→ cache DNS local (si está, responde)
→ servidor DNS recursivo (ISP)
→ Root Server ("pregunta a .com")
→ servidor .com ("pregunta al autoritativo de google.com")
→ servidor autoritativo responde la IP
→ la respuesta regresa al usuario
```

### Tipos de registros

|Registro|Descripción|Ejemplo|
|---|---|---|
|A|IPv4|google.com → 142.250.65.206|
|AAAA|IPv6|google.com → 2001:4860:4860::8888|
|CNAME|Alias (nombre canónico)|www.google.com → google.com|
|MX|Servidor de correo|google.com → mail.google.com|
|NS|Servidor DNS|google.com → ns1.google.com|
|TXT|Texto arbitrario|SPF, DKIM, verificación|
|PTR|Reverse DNS (IP → nombre)|8.8.8.8 → dns.google|
|SOA|Info de la zona DNS|—|
|SRV|Servicios específicos|`_sip._tcp.example.com`|

### Comandos

```bash
# nslookup
nslookup google.com
nslookup google.com 8.8.8.8          # servidor específico
nslookup -type=MX google.com

# dig (más detallado)
dig google.com
dig google.com MX
dig @8.8.8.8 google.com
dig google.com +short
dig google.com +trace                # traza la resolución completa
dig -x 8.8.8.8                       # reverse DNS

# host
host google.com
host -a google.com                   # todos los registros
```

### DNS Cache Poisoning

El atacante envía respuestas DNS falsas (ej. `facebook.com → IP del atacante`), el resolver las cachea, y las víctimas terminan en un sitio malicioso que roba credenciales. **Prevención:** DNSSEC, usar resolvers confiables (Google, Cloudflare), mantener los servidores DNS actualizados.

### DoH / DoT

DNS tradicional viaja en texto plano — el ISP ve cada dominio consultado. **DNS over HTTPS/TLS** cifra las consultas, mejorando privacidad y dificultando la censura. Servidores: Cloudflare `1.1.1.1`, Google `8.8.8.8`, Quad9 `9.9.9.9`.

---

## DHCP (Dynamic Host Configuration Protocol)

Asigna IP, máscara, gateway y DNS automáticamente a nuevos dispositivos.

### Proceso DORA

```
Cliente → DISCOVER (broadcast: "¿hay servidor DHCP?")
Servidor → OFFER   ("aquí una IP: 192.168.1.100")
Cliente → REQUEST  ("quiero esa IP")
Servidor → ACK     ("asignada, válida 24h")
```

**Lease time:** al expirar el arrendamiento el cliente debe renovar (típicamente al 50% del tiempo); si no renueva, pierde la IP.

### Ataques sobre DHCP

|Ataque|Mecanismo|Prevención|
|---|---|---|
|**DHCP Starvation**|El atacante genera miles de requests con MACs falsas para agotar el pool de IPs|DHCP Snooping, port security (límite de MACs por puerto), rate limiting|
|**Rogue DHCP Server**|El atacante monta su propio servidor DHCP y responde más rápido que el legítimo, asignando su propia IP como gateway/DNS → MITM y DNS poisoning|DHCP Snooping en el switch|

Herramienta de ataque de referencia: Yersinia.

---

## HTTP / HTTPS

### Métodos HTTP

|Método|Uso|
|---|---|
|GET|Obtener recurso (cargar página, imagen)|
|POST|Enviar datos (formularios, login)|
|PUT|Actualizar recurso completo|
|DELETE|Eliminar recurso|
|HEAD|GET sin body, solo headers|
|OPTIONS|Ver métodos permitidos (preflight CORS)|
|PATCH|Actualización parcial|

### Request / Response

```http
GET /index.html HTTP/1.1
Host: www.ejemplo.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml
Cookie: session_id=abc123xyz
```

```http
HTTP/1.1 200 OK
Server: Apache/2.4.41
Content-Type: text/html; charset=UTF-8
Set-Cookie: session_id=xyz789; HttpOnly; Secure

<!DOCTYPE html>...
```

### Status codes

|Rango|Categoría|Comunes|
|---|---|---|
|1xx|Informational|100 Continue|
|2xx|Success|200 OK, 201 Created|
|3xx|Redirection|301 Moved, 302 Found, 304 Not Modified|
|4xx|Client Error|400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found|
|5xx|Server Error|500 Internal Error, 502 Bad Gateway, 503 Unavailable|

### HTTPS — TLS Handshake

```
1. ClientHello       — cliente propone cifrados soportados
2. ServerHello        — servidor elige cifrado y envía su certificado
3. Cliente verifica el certificado (CA confiable, dominio, no expirado)
4. Intercambio de claves — cliente cifra una clave de sesión con la clave pública del servidor
5. Servidor descifra con su clave privada
6. Ambos comparten la clave de sesión → comienza la comunicación cifrada
```

Aporta: confidencialidad (cifrado), integridad (detecta modificación), autenticación (identidad del servidor verificada).

---

# Direccionamiento IP y Subnetting

## Clases de IPv4 (classful, histórico)

|Clase|Rango|Máscara por defecto|Hosts|Uso|
|---|---|---|---|---|
|A|1.0.0.0 – 126.255.255.255|/8 (255.0.0.0)|16.777.214|Redes grandes|
|B|128.0.0.0 – 191.255.255.255|/16 (255.255.0.0)|65.534|Redes medianas|
|C|192.0.0.0 – 223.255.255.255|/24 (255.255.255.0)|254|Redes pequeñas|
|D|224.0.0.0 – 239.255.255.255|—|—|Multicast|
|E|240.0.0.0 – 255.255.255.255|—|—|Experimental|

**Direcciones especiales:** `127.0.0.0/8` loopback (127.0.0.1 = localhost) · `0.0.0.0` "esta red" · `255.255.255.255` broadcast limitado · `169.254.0.0/16` APIPA (link-local).

## Máscaras de subred

La máscara separa una IP en porción de red y porción de host.

```
IP:   192.168.1.100  = 11000000.10101000.00000001.01100100
Mask: 255.255.255.0  = 11111111.11111111.11111111.00000000
                        └────────── Red (24) ──────┘└ Host(8)┘

IP AND Mask = Network address → 192.168.1.0
```

### CIDR

|CIDR|Máscara|Hosts utilizables|
|---|---|---|
|/8|255.0.0.0|16.777.214|
|/16|255.255.0.0|65.534|
|/24|255.255.255.0|254|
|/25|255.255.255.128|126|
|/26|255.255.255.192|62|
|/27|255.255.255.224|30|
|/28|255.255.255.240|14|
|/29|255.255.255.248|6|
|/30|255.255.255.252|2|
|/31|255.255.255.254|2 (punto a punto)|
|/32|255.255.255.255|1 (host único)|

### Fórmulas

```
Nº de subredes = 2^n        (n = bits prestados a la red)
Hosts por subred = 2^h - 2  (h = bits de host; -2 por red y broadcast)
```

### Ejemplo — Red simple /24

```
192.168.1.0/24
Red:        192.168.1.0
Primera IP: 192.168.1.1     (gateway típico)
Última IP:  192.168.1.254
Broadcast:  192.168.1.255
Hosts:      254
```

### Ejemplo — Dividir /24 en 4 subredes /26

```
192.168.1.0/26    → red .0    | rango .1–.62   | broadcast .63
192.168.1.64/26   → red .64   | rango .65–.126 | broadcast .127
192.168.1.128/26  → red .128  | rango .129–.190| broadcast .191
192.168.1.192/26  → red .192  | rango .193–.254| broadcast .255
```

### VLSM (Variable Length Subnet Mask)

Para necesidades de hosts distintas por subred, sobre `192.168.1.0/24`:

|Subred|Hosts necesarios|Máscara|Red|Rango de hosts|Broadcast|
|---|---|---|---|---|---|
|A|100|/25 (126 hosts)|192.168.1.0|.1–.126|.127|
|B|50|/26 (62 hosts)|192.168.1.128|.129–.190|.191|
|C|20|/27 (30 hosts)|192.168.1.192|.193–.222|.223|
|D|10|/28 (14 hosts)|192.168.1.224|.225–.238|.239|

### Ejercicios resueltos

**1)** `IP 10.20.30.100 / máscara 255.255.255.192 (/26)`

```
Último octeto en binario: 100 = 01100100, máscara 192 = 11000000
Red:       10.20.30.64
Primera:   10.20.30.65
Última:    10.20.30.126
Broadcast: 10.20.30.127
Hosts:     62
```

**2)** Empresa necesita 6 subredes con ≥25 hosts cada una, sobre `172.16.0.0/16`.

```
25 hosts → 2^5=32 (30 utilizables) → 5 bits de host → máscara /27
Subredes disponibles en /16 con /27: 2^(27-16) = 2048 → más que suficientes
```

---

# Routing y Switching

## Routing

Reenvía paquetes entre redes distintas según la **tabla de routing** de cada router: _"para llegar a red X, sal por interfaz Y, siguiente salto Z"_.

```bash
ip route / route -n     # Linux
route print              # Windows
show ip route             # Cisco
```

```
Destination     Gateway         Genmask         Iface
0.0.0.0         192.168.1.1     0.0.0.0         eth0   ← ruta por defecto
192.168.1.0     0.0.0.0         255.255.255.0   eth0   ← red local
10.0.0.0        192.168.1.254   255.0.0.0       eth0   ← ruta estática
```

### Tipos de rutas

|Tipo|Origen|
|---|---|
|Conectadas|Redes directamente conectadas al router|
|Estáticas|Configuradas manualmente (`ip route add 10.0.0.0/8 via 192.168.1.254`)|
|Dinámicas|Aprendidas por protocolo de routing (RIP, OSPF, EIGRP, BGP)|

### Protocolos de routing

|Protocolo|Tipo|Métrica|Notas|
|---|---|---|---|
|**RIP**|Distance Vector|Hop count|Máx. 15 saltos, actualiza cada 30s, simple pero ineficiente|
|**OSPF**|Link State|Cost (ancho de banda)|Sin límite de saltos, convergencia rápida, usado en redes grandes|
|**BGP**|Path Vector|Políticas|Exterior Gateway Protocol, el protocolo entre ISPs de Internet|

### Selección de ruta

Orden de preferencia: 1) coincidencia de máscara más específica (_longest prefix match_) → 2) Administrative Distance (AD) → 3) métrica del protocolo. **Menor AD = más confiable.**

|Protocolo|AD|
|---|---|
|Conectada|0|
|Estática|1|
|EIGRP|90|
|OSPF|110|
|RIP|120|
|EIGRP externo|170|
|Desconocido|255 (no confiar)|

---

## NAT (Network Address Translation)

Traduce IPs privadas a públicas — permite compartir una IP pública entre varios dispositivos y oculta la red interna.

|Tipo|Mecanismo|
|---|---|
|**Static NAT**|Traducción 1-a-1 fija (típico para servidores expuestos)|
|**Dynamic NAT**|Traducción desde un pool de IPs públicas|
|**PAT / NAT Overload** (el más común)|Muchas IPs privadas → 1 IP pública, diferenciadas por puerto|

```
192.168.1.10:5000 → 203.0.113.10:50001
192.168.1.20:6000 → 203.0.113.10:50002
```

**Port Forwarding:** redirige un puerto público del router a una IP/puerto interno (ej. `203.0.113.10:80 → 192.168.1.100:80` para exponer un servidor web interno).

---

## Switching

```
||Hub|Switch|
|---|---|---|
|Capa OSI|1 (Física)|2 (Enlace)|
|Reenvío|Broadcast a todos los puertos|Unicast al puerto correcto|
|Inteligencia|Ninguna|Aprende MACs|
|Colisiones|Sí|No (full-duplex)|
|Estado actual|Obsoleto|Estándar|
```

**Tabla MAC (CAM table):** el switch aprende qué MAC está en qué puerto observando el tráfico entrante; si la MAC destino es conocida reenvía por unicast, si no hace flood por todos los puertos salvo el de origen.

**Spanning Tree Protocol (STP):** en topologías con switches redundantes, detecta loops (que provocarían un _broadcast storm_ y colapsarían la red) y bloquea puertos para mantener una topología lógica sin loops, conservando la redundancia como respaldo si falla el enlace principal.