# Modelo OSI y TCP/IP

## ¿Qué es el Modelo OSI?

**OSI** (Open Systems Interconnection), creado por la ISO en 1984. Estándar para comunicación entre sistemas: divide la complejidad en capas, cada una con funciones específicas, facilitando el troubleshooting.

```
7. Aplicación      ← Usuario
8. Presentación
9. Sesión
10. Transporte
11. Red
12. Enlace de Datos
13. Física           ← Hardware
```

**Mnemotecnia:** _All People Seem To Need Data Processing_ (de la 7 a la 1).

---

## Tabla Resumen de las 7 Capas

|Capa|Nombre|Función|PDU|Dispositivos|Protocolos clave|
|---|---|---|---|---|---|
|7|Aplicación|Interfaz entre usuario y red (protocolos, no la app en sí)|Datos|— (software)|HTTP/S, FTP, SMTP, POP3, IMAP, DNS, DHCP, SSH, Telnet, SNMP|
|6|Presentación|Formato, cifrado, compresión de datos|Datos|—|SSL/TLS, JPEG/PNG, MP3/MP4, ASCII/UTF-8|
|5|Sesión|Establecer/mantener/terminar sesiones|Datos|—|NetBIOS, PPTP, RPC, sesiones SQL|
|4|Transporte|Entrega confiable/no confiable entre hosts, puertos|Segmento (TCP) / Datagrama (UDP)|—|TCP, UDP|
|3|Red|Direccionamiento lógico y routing entre redes|Paquete|Router|IP, ICMP, ARP, IGMP|
|2|Enlace de Datos|Direccionamiento físico (MAC), framing, detección de errores|Frame (Trama)|Switch, Bridge, NIC|Ethernet (802.3), subcapas LLC/MAC|
|1|Física|Transmisión de bits por el medio físico|Bits|Hub, repetidor, cables|— (voltaje, frecuencia, señal)|

---

## Capa 7 — Aplicación

Provee los protocolos que las aplicaciones usan para comunicarse por red (no es la app, es lo que hay detrás).

|Protocolo|Función|Puerto|
|---|---|---|
|HTTP/HTTPS|Web|80/443|
|FTP|Transferencia de archivos|21|
|SMTP|Email saliente|25|
|POP3|Email entrante|110|
|IMAP|Email (sincronizado)|143|
|DNS|Resolución de nombres|53|
|DHCP|Asignación automática de IP|67/68|
|SSH|Shell seguro|22|
|Telnet|Shell inseguro|23|
|SNMP|Gestión de red|161/162|

**Vulnerabilidades típicas:** SQL Injection, XSS, buffer overflow, malware, phishing.

---

## Capa 6 — Presentación

Traduce, cifra y comprime datos entre formatos.

```
Aplicación envía "Hola" → Capa 6 codifica a bytes →
(opcional) cifra con SSL/TLS → (opcional) comprime
```

**HTTPS = HTTP (capa 7) + TLS (capa 6):** el cliente cifra con TLS, HTTP estructura el mensaje, el servidor descifra con TLS y HTTP lo procesa.

---

## Capa 5 — Sesión

Establece, mantiene y termina sesiones de comunicación; sincronización y checkpoints para recuperación de fallos.

**Ejemplo — login web:** credenciales → autenticación → se crea un Session ID → se guarda en cookie → cada request lo incluye → logout termina la sesión.

**Checkpoints:** en una transferencia grande, si falla en el byte 5.500.000 con checkpoints cada 1MB, se reanuda desde el último checkpoint, no desde cero.

---

## Capa 4 — Transporte

### TCP vs UDP

|Característica|TCP|UDP|
|---|---|---|
|Conexión|Orientado a conexión|Sin conexión|
|Confiabilidad|Garantiza entrega|No garantiza entrega|
|Orden|Ordenado|Sin orden garantizado|
|Velocidad|Más lento|Más rápido|
|Overhead|Mayor (header 20+ bytes)|Menor (header 8 bytes)|
|Control de flujo/errores|Sí|Mínimo (solo checksum)|
|Uso típico|HTTP, FTP, SSH, email|DNS, streaming, VoIP, gaming|

**Por qué UDP para streaming:** si se pierde un frame, TCP espera la retransmisión (lag/buffering); UDP simplemente descarta el frame y el video continúa con un pequeño glitch.

### TCP — Three-Way Handshake

```
Cliente → SYN       → Servidor    "¿Puedo conectar?"
Cliente ← SYN-ACK    ← Servidor    "Sí, puedes"
Cliente → ACK        → Servidor    "OK, conectado"
```

### TCP — Cierre de conexión (Four-Way Handshake)

```
→ FIN   "Quiero cerrar"
← ACK   "OK"
← FIN   "Yo también cierro"
→ ACK   "OK, cerrado"
```

**Flags TCP:** `SYN` inicia conexión · `ACK` confirma recepción · `FIN` termina conexión · `RST` resetea (error) · `PSH` enviar datos inmediatamente · `URG` datos urgentes.

**Rangos de puertos:** 0–1023 well-known (HTTP:80, SSH:22) · 1024–49151 registered (MySQL:3306, RDP:3389) · 49152–65535 dinámicos/temporales.

**Vulnerabilidades de capa 4:** SYN Flood (DoS), port scanning, session hijacking, UDP Flood.

---

## Capa 3 — Red

Direccionamiento lógico (IP) y routing entre redes.

### IPv4

Formato: 32 bits en 4 octetos, ej. `192.168.1.100` (binario: `11000000.10101000.00000001.01100100`).

**Campos relevantes del header:**

|Campo|Función|
|---|---|
|**TTL**|Cada salto (router) lo decrementa en 1; al llegar a 0 el paquete se descarta — previene loops infinitos|
|**Protocol**|`1`=ICMP, `6`=TCP, `17`=UDP|

**TTL como huella de OS (útil en reconocimiento):**

|TTL observado|SO probable|
|---|---|
|64|Linux|
|128|Windows|
|255|Cisco/dispositivos de red|

```bash
ping 192.168.1.100
# TTL=64 → probablemente Linux
```

### IPv6

128 bits en 8 grupos hexadecimales: `2001:0db8:85a3:0000:0000:8a2e:0370:7334` → simplificado `2001:db8:85a3::8a2e:370:7334`.

Ventajas: 340 undecillones de direcciones, no necesita NAT, IPSec integrado, sin broadcast (usa multicast), autoconfiguración.

### ICMP

Mensajes de error y diagnóstico.

|Tipo|Significado|
|---|---|
|0|Echo Reply (respuesta a ping)|
|3|Destination Unreachable|
|5|Redirect|
|8|Echo Request (ping)|
|11|Time Exceeded (TTL=0)|

`ping` usa Echo Request/Reply. `traceroute` envía paquetes con TTL incremental: cada router que descarta el paquete por TTL=0 responde "Time Exceeded", revelando la ruta salto a salto.

### ARP

Traduce IP → MAC dentro de la red local.

```
PC: "¿Quién tiene 192.168.1.1?"           (ARP Request, broadcast FF:FF:FF:FF:FF:FF)
Router: "Soy yo, mi MAC es AA:BB:CC:DD:EE:FF"   (ARP Reply, unicast)
PC guarda: 192.168.1.1 → AA:BB:CC:DD:EE:FF   (tabla ARP)
```

```bash
arp -a          # ver tabla ARP (Linux/Mac/Windows)
```

**ARP Spoofing:** el atacante envía ARP falsos anunciando su propia MAC como la del gateway; la víctima actualiza su tabla ARP y todo su tráfico pasa por el atacante, que lo reenvía al router real → Man-in-the-Middle.

**Vulnerabilidades de capa 3:** IP Spoofing, ICMP Flood (Ping of Death), ARP Spoofing/Poisoning, ataques de enrutamiento.

---

## Capa 2 — Enlace de Datos

Transferencia entre dispositivos adyacentes en la misma red. Subcapas: **LLC** (interfaz con capa 3) y **MAC** (acceso al medio físico).

### Dirección MAC

48 bits (6 bytes) en hex, ej. `AA:BB:CC:DD:EE:FF`. Primeros 3 bytes = OUI (fabricante), últimos 3 = ID único del dispositivo.

```bash
ip link show / ifconfig     # Linux/Mac
ipconfig /all / getmac      # Windows
```

Tipos: unicast, multicast, broadcast (`FF:FF:FF:FF:FF:FF`).

### Frame Ethernet

```
[Preamble(7)] [Dest MAC(6)] [Src MAC(6)] [Type(2)] [Data(46-1500)] [FCS(4)]
```

`Type`: `0x0800`=IPv4, `0x0806`=ARP. `FCS` = CRC para detección de errores.

### Switch

Reenvía frames según la tabla MAC (CAM table): aprende qué MAC está en qué puerto observando el tráfico entrante. Si la MAC destino es conocida, reenvía por unicast a ese puerto; si no, hace flood por todos los puertos.

### VLAN

Segmenta una red física en redes lógicas independientes (aislamiento de tráfico, reduce el dominio de broadcast, mejora seguridad y rendimiento).

```bash
# Cisco — configuración básica
enable
configure terminal
vlan 10
name Ventas
exit
interface FastEthernet0/1
switchport mode access
switchport access vlan 10
```

**Vulnerabilidades de capa 2:** MAC Flooding (satura la tabla CAM), ARP Spoofing, VLAN Hopping, MAC Spoofing, switch spoofing.

---

## Capa 1 — Física

Transmisión de bits como señales eléctricas, ópticas o de radio.

|Medio|Notas|
|---|---|
|UTP (cobre)|Cat5e (100Mbps-1Gbps), Cat6 (1-10Gbps), Cat6a (10Gbps); máx. 100m|
|Coaxial|Cable TV/Internet, más inmune a interferencias que UTP|
|Fibra óptica|Pulsos de luz, muy rápida, inmune a interferencia electromagnética; monomodo (largas distancias) vs multimodo (cortas)|
|Inalámbrico|WiFi (802.11), Bluetooth, celular (4G/5G), satélite|

**Topologías:** bus (antigua, Ethernet 10Base2), estrella (la más común, switch central), anillo (Token Ring, FDDI), malla (redundancia total).

**Dispositivos:** hub (broadcast a todos los puertos), repetidor (amplifica señal), cables/conectores.

**Vulnerabilidades de capa 1:** sniffing físico (acceso al cable), jamming (interferencia RF), corte de cables, acceso físico no autorizado.

---

# Modelo TCP/IP

Creado por DARPA en los 70, base de Internet. Modelo práctico de 4 capas (frente a las 7 teóricas de OSI).

## Equivalencia OSI ↔ TCP/IP

|OSI|TCP/IP|
|---|---|
|7. Aplicación / 6. Presentación / 5. Sesión|4. Aplicación|
|4. Transporte|3. Transporte|
|3. Red|2. Internet|
|2. Enlace de Datos / 1. Física|1. Acceso a Red|

|Capa TCP/IP|Protocolos|
|---|---|
|Aplicación|HTTP/S, FTP/SFTP, SMTP/POP3/IMAP, DNS, DHCP, SSH, Telnet, SNMP, NTP|
|Transporte|TCP, UDP|
|Internet|IP (v4/v6), ICMP, ARP, IGMP|
|Acceso a Red|Ethernet (802.3), WiFi (802.11), PPP, Frame Relay|

---

## Encapsulación de Datos

**Al enviar:**

```
Aplicación:    "Hola Mundo"
Transporte:    [TCP Header | Hola Mundo]              → Segmento
Internet:      [IP | TCP | Hola Mundo]                → Paquete
Acceso a Red:  [Eth | IP | TCP | Hola Mundo | FCS]     → Frame
                            ↓
                    Bits transmitidos
```

**Al recibir (desencapsulación), el proceso inverso:**

```
Bits → Frame (quita header Ethernet) → Paquete
     → Paquete (quita header IP) → Segmento
     → Segmento (quita header TCP) → Datos
     → "Hola Mundo" entregado a la aplicación
```