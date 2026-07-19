# Fundamentos de Mobile Security

## Arquitectura Móvil

### **Android Architecture:**

```
LAYERS:

Linux Kernel
├─ Hardware drivers
├─ Memory management
└─ Security (SELinux)

Hardware Abstraction Layer (HAL)
├─ Camera, GPS, Bluetooth APIs

Android Runtime (ART)
├─ Dex bytecode execution
└─ Garbage collection

Native C/C++ Libraries
├─ libc, SSL, SQLite

Java API Framework
├─ Activity Manager
├─ Content Providers
└─ Package Manager

Applications
└─ User-installed apps
```

---

### **iOS Architecture:**

```
LAYERS:

Core OS
├─ Kernel (XNU - hybrid kernel)
├─ Drivers
└─ Low-level Unix

Core Services
├─ Security, Keychain
├─ Core Foundation
└─ SQLite

Media Layer
├─ Graphics, Audio, Video

Cocoa Touch
├─ UIKit
├─ MapKit
└─ Game Center

Applications
```

---

## OWASP Mobile Top 10 (2024)

```
M1: IMPROPER PLATFORM USAGE
   • Misuso de APIs del OS
   • Permisos innecesarios

M2: INSECURE DATA STORAGE
   • Datos sensibles sin cifrar
   • Logs con información confidencial
   • Shared preferences inseguras

M3: INSECURE COMMUNICATION
   • HTTP en vez de HTTPS
   • Certificate pinning ausente
   • Weak SSL/TLS

M4: INSECURE AUTHENTICATION
   • Autenticación débil
   • Sesiones sin timeout
   • Biometría mal implementada

M5: INSUFFICIENT CRYPTOGRAPHY
   • Algoritmos débiles
   • Keys hardcodeadas
   • Custom crypto (malo)

M6: INSECURE AUTHORIZATION
   • Privilege escalation
   • Falta de validación server-side

M7: CLIENT CODE QUALITY
   • Buffer overflows
   • Format string vulnerabilities

M8: CODE TAMPERING
   • Falta de protección contra modificación
   • Sin detección de root/jailbreak

M9: REVERSE ENGINEERING
   • Código fácil de decompile
   • Strings hardcodeadas
   • Sin obfuscación

M10: EXTRANEOUS FUNCTIONALITY
   • Backdoors
   • Debug code en producción
   • Test endpoints
```

---

# Android Pentesting

## Setup de Laboratorio

### **Instalación de Herramientas:**

```bash
# Android SDK Platform Tools
wget https://dl.google.com/android/repository/platform-tools-latest-linux.zip
unzip platform-tools-latest-linux.zip
export PATH=$PATH:~/platform-tools

# ADB (Android Debug Bridge)
adb version

# Instalar Java
sudo apt install default-jdk

# APKTool
wget https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool
wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.9.1.jar
chmod +x apktool apktool_2.9.1.jar
sudo mv apktool apktool_2.9.1.jar /usr/local/bin/

# Jadx (Dex to Java decompiler)
wget https://github.com/skylot/jadx/releases/download/v1.4.7/jadx-1.4.7.zip
unzip jadx-1.4.7.zip -d jadx
export PATH=$PATH:~/jadx/bin

# MobSF (Mobile Security Framework)
docker pull opensecurity/mobile-security-framework-mobsf
docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
# http://localhost:8000
```

---

## Emulador Android

### **Genymotion (Recomendado):**

```bash
# Descargar de genymotion.com
# Instalar
chmod +x genymotion-3.5.0-linux_x64.bin
./genymotion-3.5.0-linux_x64.bin

# Crear dispositivo virtual
# Android 11 o 12 (sin Google Play)
```

---

### **Android Studio AVD:**

```bash
# Descargar Android Studio
# Tools > AVD Manager
# Create Virtual Device
# Pixel 5, Android 12

# Iniciar emulador
emulator -avd Pixel_5_API_31
```

---

## Análisis Estático

### **Decompilación con APKTool:**

```bash
# Descargar APK (ejemplo)
# De dispositivo:
adb pull /data/app/com.example.app-1/base.apk app.apk

# Decompile
apktool d app.apk -o app_decompiled

# Estructura:
app_decompiled/
├── AndroidManifest.xml  # Permisos, componentes
├── smali/               # Código en Smali
├── res/                 # Recursos
├── lib/                 # Librerías nativas
└── assets/              # Assets

# Ver AndroidManifest
cat app_decompiled/AndroidManifest.xml

# Buscar permisos peligrosos
grep "android.permission" AndroidManifest.xml
```

---

### **Decompilación a Java con Jadx:**

```bash
# GUI
jadx-gui app.apk

# CLI
jadx app.apk -d app_decompiled_java

# Código Java legible
# Buscar:
cat app_decompiled_java/**/*.java | grep -i "password\|secret\|api"

# URLs hardcodeadas
cat app_decompiled_java/**/*.java | grep -oP 'https?://[^"]*'
```

---

### **MobSF - Análisis Automatizado:**

```bash
# Subir APK a MobSF
# http://localhost:8000

# Analiza:
• Permisos
• Activities, Services, Receivers
• Network security
• Code analysis (SAST)
• Strings hardcodeadas
• URLs, IPs
• Secrets potenciales
• Malware indicators

# Score de seguridad: 0-100
```

---

## Análisis Dinámico

### **Conectar con ADB:**

```bash
# Verificar dispositivo
adb devices

# Shell en dispositivo
adb shell

# Instalar APK
adb install app.apk

# Desinstalar
adb uninstall com.example.app

# Ver logs en tiempo real
adb logcat | grep "com.example.app"

# Filtrar por tag
adb logcat -s "TAG:D"

# Limpiar logs
adb logcat -c
```

---

### **Interceptar Tráfico con Burp Suite:**

```bash
# 1. Configurar Burp proxy (8080)

# 2. Instalar certificado en Android
# Burp > Proxy > Options > Import/Export CA cert
# Exportar como DER

# Convertir a PEM
openssl x509 -inform DER -in cacert.der -out cacert.pem

# Hash del certificado
openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1
# Resultado: 9a5ba575

# Renombrar
mv cacert.pem 9a5ba575.0

# Instalar en Android (root required)
adb root
adb remount
adb push 9a5ba575.0 /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/9a5ba575.0
adb reboot

# 3. Configurar proxy en Android
# Settings > Wi-Fi > Long press network > Modify
# Proxy: Manual
# Hostname: [IP de tu máquina]
# Port: 8080

# 4. Ver tráfico en Burp
```

---

### **Frida (Dynamic Instrumentation):**

```bash
# Instalar Frida
pip3 install frida-tools

# Descargar frida-server para Android
wget https://github.com/frida/frida/releases/download/16.1.4/frida-server-16.1.4-android-arm64.xz
unxz frida-server-16.1.4-android-arm64.xz

# Push a dispositivo
adb push frida-server-16.1.4-android-arm64 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &

# Verificar
frida-ps -U

# Listar apps
frida-ps -Ua

# Hooking básico
frida -U -n "com.example.app" -l script.js
```

---

### **Frida Script - Bypass SSL Pinning:**

```javascript
// ssl-pinning-bypass.js
Java.perform(function() {
    console.log("Bypass SSL Pinning");

    // OkHttp3
    var CertificatePinner = Java.use("okhttp3.CertificatePinner");
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {
        console.log("SSL Pinning bypassed for: " + arguments[0]);
        return;
    };

    // TrustManager
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    TrustManager.checkServerTrusted.implementation = function() {
        console.log("checkServerTrusted bypassed");
    };

    console.log("Bypass complete!");
});

// Ejecutar
// frida -U -f com.example.app -l ssl-pinning-bypass.js --no-pause
```

---

### **Frida Script - Hook Métodos:**

```javascript
// hook-login.js
Java.perform(function() {
    var Activity = Java.use("com.example.app.LoginActivity");

    Activity.login.implementation = function(username, password) {
        console.log("[+] Login called!");
        console.log("[+] Username: " + username);
        console.log("[+] Password: " + password);

        // Llamar al método original
        return this.login(username, password);
    };
});

// Uso
// frida -U -n "com.example.app" -l hook-login.js
```

---

## Bypass de Protecciones

### **Root Detection Bypass:**

```javascript
// root-detection-bypass.js
Java.perform(function() {
    // Bypass común
    var RootDetection = Java.use("com.example.app.RootDetection");
    RootDetection.isRooted.implementation = function() {
        console.log("Root check bypassed");
        return false;
    };

    // Bypass file check
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.includes("su") || path.includes("magisk")) {
            console.log("Hiding: " + path);
            return false;
        }
        return this.exists();
    };
});
```

---

### **Emulator Detection Bypass:**

```bash
# Modificar build.prop
adb root
adb shell
mount -o rw,remount /system
nano /system/build.prop

# Cambiar:
ro.product.manufacturer=Samsung
ro.product.model=SM-G975F
ro.build.fingerprint=samsung/...

# Restart
reboot
```

---

## Vulnerabilidades Comunes

### **Insecure Data Storage:**

```bash
# Shared Preferences (XML)
adb shell
cd /data/data/com.example.app/shared_prefs
cat *.xml

# SQLite databases
cd /data/data/com.example.app/databases
sqlite3 userdata.db
.tables
SELECT * FROM users;

# Internal storage
cd /data/data/com.example.app/files
cat sensitive_data.txt

# SD card (world-readable)
cd /sdcard/Android/data/com.example.app/
```

---

### **Insecure Communication:**

```bash
# Buscar HTTP (no HTTPS)
grep -r "http://" app_decompiled_java/

# Verificar cleartext traffic permitido
cat AndroidManifest.xml | grep "cleartextTrafficPermitted"
# Si es true → vulnerable

# Network Security Config
cat res/xml/network_security_config.xml
```

---

### **Intent Injection:**

```bash
# Exportar activity vulnerable
adb shell am start -n com.example.app/.PrivateActivity

# Con datos
adb shell am start -n com.example.app/.PrivateActivity --es "data" "malicious"

# Broadcast receiver
adb shell am broadcast -a com.example.app.ACTION_CUSTOM --es "cmd" "hack"
```

---

### **WebView Vulnerabilities:**

```javascript
// Buscar WebView con JavaScript enabled
// En código decompilado:
webView.getSettings().setJavaScriptEnabled(true);
webView.addJavascriptInterface(new WebAppInterface(), "Android");

// Si addJavascriptInterface → RCE posible
// XSS en WebView → acceso a bridge
```