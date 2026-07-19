# iOS Pentesting

## iOS Security Basics

### **Diferencias con Android:**

```
SANDBOXING:
• Más estricto que Android
• Apps aisladas completamente
• Compartir datos solo via APIs autorizadas

CODE SIGNING:
• Toda app debe estar firmada
• Verificación en cada ejecución
• Sin firma válida = no ejecuta

KEYCHAIN:
• Almacenamiento seguro de credenciales
• Cifrado por hardware

APP STORE:
• Review process estricto
• Menor malware que Android
• Sideloading difícil (sin jailbreak)
```

---

## Jailbreak

### **¿Qué es Jailbreak?**

```
Obtener acceso root en iOS

TIPOS:
• Untethered: Permanente (mejor)
• Semi-untethered: Requiere re-jailbreak después de reboot
• Tethered: Requiere PC para boot

HERRAMIENTAS:
• checkra1n (iOS 12-14.8)
• unc0ver (iOS 11-14.8)
• Palera1n (iOS 15-16)
• Dopamine (iOS 15-16)

NOTA: Jailbreak es necesario para pentesting profundo
```

---

## Setup iOS Pentesting

### **Herramientas Básicas:**

```bash
# Instalar en Mac/Linux
# Xcode (Mac only)
# libimobiledevice
brew install libimobiledevice
# O
sudo apt install libimobiledevice-utils

# ideviceinstaller
brew install ideviceinstaller

# Conectar iPhone vía USB
idevice_id -l

# Info del dispositivo
ideviceinfo

# Screenshot
idevicescreenshot screenshot.png

# Logs
idevicesyslog
```

---

### **SSH a iPhone (con Jailbreak):**

```bash
# Instalar OpenSSH en iPhone (Cydia)

# Conectar (default password: alpine)
ssh root@[iphone-ip]

# CAMBIAR PASSWORD inmediatamente
passwd

# Acceso completo al filesystem
ls /var/mobile/Containers/Data/Application/
```

---

## Análisis de IPA

### **Obtener IPA:**

```bash
# Desde dispositivo jailbroken
# Instalar Clutch o frida-ios-dump

# Con frida-ios-dump
git clone https://github.com/AloneMonkey/frida-ios-dump
cd frida-ios-dump
pip3 install -r requirements.txt

# Listar apps
python3 dump.py -l

# Dump de app
python3 dump.py com.example.app

# Genera .ipa file

# Analizar IPA (es un ZIP)
unzip app.ipa -d app_extracted

# Estructura:
Payload/
  App.app/
    Info.plist         # Configuración
    executable         # Binary
    embedded.mobileprovision
    Frameworks/
    Assets.car
```

---

### **Análisis Estático iOS:**

```bash
# Ver Info.plist
plutil -p Payload/App.app/Info.plist

# Buscar permisos
grep -i "NSLocationWhenInUseUsageDescription" Info.plist

# Strings del binary
strings Payload/App.app/App | grep "http"
strings Payload/App.app/App | grep -i "password"

# Classdump (ver clases Objective-C)
brew install class-dump
class-dump Payload/App.app/App > app_classes.txt

# Ver métodos de clase específica
grep -A 20 "@interface LoginViewController" app_classes.txt
```

---

### **MobSF para iOS:**

```bash
# Subir .ipa a MobSF
# http://localhost:8000

# Análisis similar a Android:
• Binary analysis
• Plist analysis
• Code review
• Security issues
```

---

## Frida en iOS

### **Setup:**

```bash
# Instalar frida en dispositivo (Cydia)
# Agregar repo: https://build.frida.re
# Instalar "Frida"

# Verificar desde PC
frida-ps -U

# Listar apps
frida-ps -Ua
```

---

### **Bypass Jailbreak Detection:**

```javascript
// ios-jailbreak-bypass.js
if (ObjC.available) {
    // Bypass file check
    var NSFileManager = ObjC.classes.NSFileManager;
    NSFileManager['- fileExistsAtPath:'].implementation = function(path) {
        var response = this.fileExistsAtPath_(path);
        if (path.toString().includes("cydia") || 
            path.toString().includes("mobile substrate")) {
            console.log("Hiding: " + path);
            return false;
        }
        return response;
    };

    console.log("[+] Jailbreak detection bypassed");
}

// Ejecutar
// frida -U -f com.example.app -l ios-jailbreak-bypass.js --no-pause
```

---

### **Hook Métodos iOS:**

```javascript
// hook-ios-method.js
if (ObjC.available) {
    var LoginViewController = ObjC.classes.LoginViewController;

    LoginViewController['- loginWithUsername:password:'].implementation = function(username, password) {
        console.log("[+] Login called");
        console.log("[+] Username: " + username);
        console.log("[+] Password: " + password);

        return this.loginWithUsername_password_(username, password);
    };
}
```

---

## Vulnerabilidades iOS

### **Insecure Data Storage:**

```bash
# SSH a dispositivo
ssh root@iphone-ip

# App data location
cd /var/mobile/Containers/Data/Application/[UUID]/

# Documentos
cat Documents/sensitive_data.txt

# Preferencias
cd Library/Preferences
plutil -p com.example.app.plist

# Keychain (requiere herramientas especiales)
# Keychain-dumper
./keychain_dumper > keychain.txt
```

---

# IoT Security

## Introducción a IoT

### **Componentes IoT:**

```
DISPOSITIVOS:
• Sensores (temperatura, movimiento)
• Actuadores (luces, cerraduras)
• Smart home (Alexa, Google Home)
• Wearables (smartwatches)
• Cámaras IP
• Routers

PROTOCOLOS:
• MQTT (Message Queue Telemetry Transport)
• CoAP (Constrained Application Protocol)
• Zigbee
• Z-Wave
• Bluetooth Low Energy (BLE)
• LoRaWAN

VULNERABILIDADES COMUNES:
• Credenciales por defecto
• Sin cifrado
• Firmware vulnerable
• Actualizaciones inexistentes
• Backdoors
```

---

## Reconocimiento IoT

### **Shodan (Motor de búsqueda IoT):**

```bash
# Buscar cámaras expuestas
shodan search "webcam"
shodan search "default password"

# Dispositivos específicos
shodan search "port:1883"  # MQTT
shodan search "port:5683"  # CoAP

# Por país
shodan search "country:US camera"

# Con API
shodan download --limit 1000 camera.json.gz "webcam"
```

---

### **Censys:**

```
https://search.censys.io/

Similar a Shodan:
• Escanea internet
• Encuentra dispositivos expuestos
• Certificados SSL
• Servicios abiertos
```

---

## Ataques a Protocolos IoT

### **MQTT (Port 1883, 8883):**

```bash
# Instalar mosquitto clients
sudo apt install mosquitto-clients

# Connect a broker público (test)
mosquitto_sub -h test.mosquitto.org -t "#" -v

# Subscribirse a todos los topics
mosquitto_sub -h [target-broker] -t "#" -v

# Sin autenticación (común)
mosquitto_sub -h [target-ip] -t "#" -v

# Publicar mensaje
mosquitto_pub -h [target-ip] -t "home/lights" -m "ON"

# Con credenciales
mosquitto_sub -h [target-ip] -t "#" -u username -P password -v

# SSL/TLS (8883)
mosquitto_sub -h [target-ip] -p 8883 -t "#" --cafile ca.crt
```

---

### **CoAP (Port 5683 UDP):**

```bash
# Instalar libcoap
sudo apt install libcoap-bin

# Get resource
coap-client -m get coap://[target-ip]/.well-known/core

# Observar cambios
coap-client -m get -s 10 coap://[target-ip]/sensor/temp

# POST
echo "data" | coap-client -m post coap://[target-ip]/actuator
```

---

### **Bluetooth Low Energy (BLE):**

```bash
# Escanear dispositivos BLE
sudo hcitool lescan

# Info de dispositivo
sudo gatttool -b [MAC] -I
> connect
> primary   # Listar servicios
> characteristics  # Listar características

# Leer característica
> char-read-hnd 0x0011

# Escribir
> char-write-req 0x0011 0100

# BLE MITM con btlejack
sudo pip3 install btlejack
btlejack -s
btlejack -f [MAC1] -f [MAC2]
```

---

## Hardware Hacking

### **UART (Universal Asynchronous Receiver-Transmitter):**

```bash
# Identificar pines UART en PCB
# Buscar: TX, RX, GND, VCC (a veces)

# Usar USB-to-TTL adapter (CP2102, FTDI)
# Conectar:
# GND → GND
# RX → TX del dispositivo
# TX → RX del dispositivo

# Determinar baudrate
# Común: 9600, 19200, 38400, 57600, 115200

# Conectar con screen
screen /dev/ttyUSB0 115200

# O minicom
minicom -D /dev/ttyUSB0 -b 115200

# Si obtienes acceso → shell root común
```

---

### **JTAG (Joint Test Action Group):**

```bash
# Depuración y programación de chips

# Identificar pines JTAG
# TDI, TDO, TMS, TCK, GND

# Usar OpenOCD
sudo apt install openocd

# Configuración
openocd -f interface/ftdi/um232h.cfg -f target/stm32f1x.cfg

# Dump firmware
> dump_image firmware.bin 0x08000000 0x10000

# Debug con GDB
arm-none-eabi-gdb
> target remote localhost:3333
```

---

### **SPI (Serial Peripheral Interface):**

```bash
# Flash memory chips (EEPROM, NOR Flash)

# Usar flashrom o Bus Pirate
flashrom -p buspirate_spi:dev=/dev/ttyUSB0

# Leer flash
flashrom -p buspirate_spi:dev=/dev/ttyUSB0 -r firmware.bin

# Analizar firmware
binwalk firmware.bin

# Extraer filesystem
binwalk -e firmware.bin
```

---

## Análisis de Firmware

### **Binwalk:**

```bash
# Instalar
sudo apt install binwalk

# Escanear firmware
binwalk firmware.bin

# Extraer filesystems
binwalk -e firmware.bin

# Se extrae a _firmware.bin.extracted/

# Buscar credenciales
grep -r "password" _firmware.bin.extracted/
grep -r "root:" _firmware.bin.extracted/etc/

# Buscar backdoors
grep -r "backdoor\|debug" _firmware.bin.extracted/
```

---

### **Firmware Analysis Toolkit (FAT):**

```bash
# Instalar
git clone https://github.com/attify/firmware-analysis-toolkit
cd firmware-analysis-toolkit
./setup.sh

# Analizar firmware
./fat.py firmware.bin

# Emula firmware en QEMU
# Acceso a shell emulado
# Permite pentesting sin dispositivo físico
```

---

## Ataques a Dispositivos IoT

### **Cámaras IP:**

```bash
# Credenciales por defecto
admin:admin
admin:12345
admin:(blank)
root:root

# Buscar exploits conocidos
searchsploit "IP Camera"

# RTSP stream (port 554)
ffplay rtsp://admin:password@[ip]:554/stream

# Onvif (discovery)
onvif-gui
```

---

### **Smart Plugs:**

```python
# Ejemplo: TP-Link Kasa smart plug
from pyHS100 import SmartPlug

plug = SmartPlug("192.168.1.100")
print(plug.state)
plug.turn_on()
plug.turn_off()

# Vulnerable a MITM si no usa cifrado
```

---

# Defensa Móvil e IoT

## Seguridad Móvil

### **Desarrollo Seguro:**

```
ANDROID:
• Usar ProGuard/R8 para ofuscación
• Implementar certificate pinning
• Cifrar datos sensibles (AES-256)
• No hardcodear secrets
• Verificar firma de APK
• Detectar root (pero asume bypass)
• HTTPS siempre
• Validar inputs

iOS:
• App Transport Security habilitado
• Keychain para credenciales
• Ofuscación de código
• Detectar jailbreak
• Certificate pinning
• No logs en producción
```

---

### **MDM (Mobile Device Management):**

```
• Gestión centralizada
• Políticas de seguridad
• Remote wipe
• App whitelist/blacklist
• Segregación de datos (BYOD)

Soluciones:
• Microsoft Intune
• VMware Workspace ONE
• MobileIron
• Jamf (iOS)
```

---

## Seguridad IoT

### **Best Practices:**

```
✅ CAMBIAR CREDENCIALES DEFAULT
✅ Actualizar firmware regularmente
✅ Deshabilitar servicios innecesarios
✅ Usar VLAN separada para IoT
✅ Firewall/IDS para monitorear
✅ Cifrado en comunicaciones (TLS)
✅ Autenticación fuerte
✅ Principio de mínimo privilegio
✅ Logs y monitoreo
✅ Considerar seguridad física
```

---

### **Network Segmentation:**

```
HOME NETWORK SEGURO:

VLAN 10: Trusted (PCs, phones)
VLAN 20: IoT (cámaras, smart home)
VLAN 30: Guest

Firewall rules:
• IoT → Internet (permitir)
• IoT → Trusted (denegar)
• Trusted → IoT (permitir solo puertos necesarios)
• Guest → aislado
```