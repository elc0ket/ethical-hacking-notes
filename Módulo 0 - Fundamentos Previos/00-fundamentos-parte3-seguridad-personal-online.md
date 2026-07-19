## Introducción

Conjunto de prácticas y herramientas que protegen información, identidad y privacidad en Internet: proteger datos personales/financieros, prevenir robo de identidad, evitar fraudes, mantener privacidad y reputación online.

**Amenazas comunes:** phishing, malware (virus, ransomware), robo de identidad, ingeniería social, violaciones de datos (breaches), suplantación de identidad, tracking online.

---

## Contraseñas Seguras

### Características de una contraseña fuerte

- **Longitud:** mínimo 12 caracteres (ideal 16+)
- **Complejidad:** mayúsculas, minúsculas, números, símbolos
- **Impredecible:** sin palabras comunes ni información personal
- **Única:** distinta para cada servicio

|Tipo|Ejemplo|Resistencia|
|---|---|---|
|Muy débil|`password`|< 1 segundo|
|Débil|`Password1`|< 1 minuto|
|Media|`Pass123!`|horas/días|
|Fuerte|`M!p@ssw0rd#2024`|meses|
|Muy fuerte|`Tr3$-C0mp1ej0_P@ssw0rd&2024!`|años|

### Métodos para crearlas

**Frase memorable → contraseña:**

```
"Mi primer coche fue un Honda Civic del 2010" → MpcfuHCd2010!
```

**Acrónimo de una frase:**

```
"En la casa de mi abuela siempre hay 5 gatos corriendo" → Elcdmash5gc!
```

**Generador aleatorio:** `x7#Kp9@mL2$qW5nR`

### Nunca usar

`123456`, `password`, `qwerty`, nombre+fecha de nacimiento, nombre de mascota o familiares, palabras de diccionario, secuencias (`abcd`, `1234`), información pública de redes sociales.

---

## Gestores de Contraseñas

Almacenan y gestionan credenciales de forma cifrada; solo hay que recordar una contraseña maestra. Generan contraseñas fuertes, autorrellenan y sincronizan entre dispositivos.

|Gestor|Tipo|Notas|
|---|---|---|
|**Bitwarden**|Open source|Gratis/Premium — recomendado para empezar|
|**1Password**|Comercial|~$35/año, interfaz pulida|
|**LastPass**|Freemium|Popular, en la nube|
|**KeePass**|Open source|Gratis, almacenamiento local|
|**Dashlane**|Comercial|Incluye VPN|
|**NordPass**|Comercial|De los creadores de NordVPN|

**Flujo de configuración:** instalar → crear cuenta con contraseña maestra fuerte → instalar extensión de navegador → importar contraseñas existentes → generar nuevas → habilitar 2FA en el propio gestor.

---

## Autenticación de Dos Factores (2FA)

Añade un segundo factor más allá de la contraseña: código SMS, app TOTP, llave física, biometría.

|Tipo|Seguridad|
|---|---|
|SMS|Media (interceptable/SIM swapping)|
|App (TOTP — Google Authenticator, Authy)|Alta|
|Push notification|Alta|
|Hardware key (YubiKey)|Muy alta|
|Biométrica|Alta|

**Apps recomendadas:** Google Authenticator (simple), Authy (backup en nube, multi-dispositivo), Microsoft Authenticator (integración MS), Aegis (Android, open source, sin conexión).

**Servicios que deben tener 2FA activado:** correo, redes sociales, banca online, PayPal/pagos, cloud storage, gestor de contraseñas, cuentas de trabajo, cuentas de gaming.

---

## Phishing

Ataque donde se suplanta a una entidad legítima para robar información.

**Tipos:** email phishing, spear phishing (dirigido), smishing (SMS), vishing (llamadas), whaling (dirigido a ejecutivos).

### Señales de alerta

|Señal|Ejemplo|
|---|---|
|Remitente falsificado|`support@paypa1.com` (1 en vez de l), `support@paypal-secure.net`|
|Urgencia/amenaza|"Tu cuenta será suspendida en 24 horas"|
|Enlace con dominio distinto al mostrado|Texto "Ir a PayPal" → URL real `paypa1-secure.ru/login.php`|
|Errores gramaticales/genéricos|"Estimado cliente," faltas de ortografía|
|Adjuntos sospechosos|`factura.pdf.exe` (doble extensión), `.scr`, `.bat`, `.cmd`|

### Cómo protegerse

Verificar el remitente real · no hacer clic en enlaces sospechosos (escribir la URL directamente) · comprobar HTTPS/candado · no descargar adjuntos de fuentes desconocidas · usar 2FA en todo lo importante · reportar como spam.

---

## Privacidad en Redes Sociales

|Red|Configuración recomendada|
|---|---|
|**Facebook**|Publicaciones futuras → Amigos · Lista de amigos → Solo yo · Desactivar reconocimiento facial · Revisar apps con acceso|
|**Instagram**|Cuenta privada ON · Ocultar historias a personas concretas · Desactivar "Visto por última vez"|
|**Twitter/X**|Proteger Tweets · Desactivar etiquetado de ubicación · Habilitar 2FA|

**Nunca compartir:** dirección completa, teléfono personal, fotos de documentos de identidad, datos de tarjetas, planes de viaje antes de viajar, ubicación en tiempo real, información laboral sensible, rutinas predecibles, fotos de llaves/códigos, datos de menores.

**Buenas prácticas:** revisar privacidad cada 3 meses, listas personalizadas de audiencia, pensar antes de publicar, no aceptar desconocidos, desactivar geolocalización en fotos, usuarios distintos por red, buscar el propio nombre en Google periódicamente.

---

## Navegación Segura

### HTTPS

`https://` (candado 🔒) vs `http://` (sin cifrar). Verificar siempre el candado y el certificado — **un sitio malicioso también puede tener HTTPS**, revisar igualmente el dominio.

### Extensiones recomendadas

|Extensión|Función|
|---|---|
|uBlock Origin|Bloqueador de anuncios|
|Privacy Badger|Bloquea rastreadores|
|HTTPS Everywhere|Fuerza conexiones HTTPS|
|Bitwarden|Gestor de contraseñas|
|ClearURLs|Elimina parámetros de rastreo|

### Modo incógnito/privado

`Ctrl+Shift+N` (Chrome/Edge) · `Ctrl+Shift+P` (Firefox).

No guarda historial, cookies ni datos de formularios — **pero no anonimiza**: el ISP, los sitios web y el empleador/escuela aún pueden verte.

### VPN

Cifra el tráfico y oculta la IP real (`PC → VPN → Internet`).

**Usar cuando:** WiFi público, evitar censura geográfica, proteger privacidad frente al ISP, contenido regional.

**Recomendadas:** NordVPN, ProtonVPN (plan gratis), Mullvad, IVPN. Evitar VPNs gratuitas sin reputación (pueden vender los datos).

### Tor Browser

Anonimiza el tráfico en múltiples nodos (`PC → Nodo1 → Nodo2 → Nodo3 → Internet`, cifrado en cada salto). Descarga: torproject.org.

Ventajas: máximo anonimato, acceso a `.onion`, gratis y open source. Desventajas: lento, algunos sitios lo bloquean, no apto para descargas grandes.

---

## Correo Electrónico Seguro

|Servicio|Cifrado|Sede|
|---|---|---|
|ProtonMail|E2E|Suiza|
|Tutanota|E2E|Alemania|
|Mailfence|Opcional|Bélgica|
|StartMail|PGP|Países Bajos|

E2E = cifrado de extremo a extremo.

**Gmail — configuración segura:** verificación en 2 pasos activada · revisar sesiones activas y cerrar las sospechosas · revisar permisos de apps de terceros en Google Account → Seguridad y revocar innecesarios · usar contraseñas de aplicación para apps sin soporte 2FA.

**Correos desechables** (para registros no importantes): Temp-Mail.org, Guerrilla Mail, 10 Minute Mail, Maildrop.

---

## Verificar Filtraciones de Datos

**Have I Been Pwned** (haveibeenpwned.com): introducir el email y comprobar si aparece en alguna brecha conocida.

**Si los datos aparecieron filtrados:** cambiar la contraseña inmediatamente → activar 2FA si no lo tenía → revisar actividad reciente sospechosa → cambiar la contraseña en cualquier otro sitio donde se reutilizara → monitorear la cuenta.

---

## Seguridad Financiera Online

**Compras online:** verificar HTTPS/candado, usar tarjetas virtuales cuando sea posible, revisar reputación del vendedor, usar pasarelas de pago seguras (PayPal, etc.), no guardar tarjetas en los sitios, revisar extractos regularmente, configurar alertas de transacciones.

**Tarjetas virtuales:** número distinto al de la tarjeta real, límite configurable, expiración corta, desechables tras el uso — si se filtran, no afectan a la tarjeta real.

**Banca móvil:** descargar solo desde tiendas oficiales, verificar que sea la app oficial del banco, habilitar biometría, no usar en WiFi público sin VPN, mantener la app actualizada, no usar en dispositivos con jailbreak/root.

---

## Seguridad en Dispositivos Móviles

```
||Android|iOS|
|---|---|---|
|Bloqueo|PIN/patrón/huella + cifrado de dispositivo|Face ID / Touch ID|
|Protección|Google Play Protect activado|Revisar permisos y desactivar rastreo de apps en Privacidad|
|Backup|—|iCloud: "Buscar mi iPhone" + backup cifrado|
|Instalación|Solo Play Store oficial|Solo App Store oficial|
```


**Peligros a evitar:** APKs de tiendas no oficiales, jailbreak/root, WiFi público sin VPN, permisos excesivos a apps, sistema operativo sin actualizar, enlaces en SMS desconocidos.

---

## Checklist de Seguridad Personal

**Hoy:**

- [ ] Instalar gestor de contraseñas
- [ ] Activar 2FA en el correo principal
- [ ] Verificar en haveibeenpwned.com
- [ ] Poner en privado las redes sociales
- [ ] Instalar uBlock Origin
- [ ] Revisar permisos de apps en el móvil

**Esta semana:**

- [ ] Cambiar contraseñas débiles (top 5 cuentas)
- [ ] Activar 2FA en redes sociales y servicios financieros
- [ ] Revisar configuración de privacidad en todas las redes
- [ ] Configurar alertas de transacciones bancarias
- [ ] Limpiar apps con acceso a cuentas

**Mensual:**

- [ ] Revisar extractos bancarios
- [ ] Revisar sesiones activas en cuentas importantes
- [ ] Actualizar apps y sistema operativo
- [ ] Revisar configuración de privacidad
- [ ] Backup de datos importantes

**Anual:**

- [ ] Cambiar contraseñas importantes
- [ ] Revisar toda la configuración de privacidad
- [ ] Borrar cuentas sin uso
- [ ] Revisar qué información hay de uno mismo en Google
- [ ] Actualizar códigos de respaldo de 2FA

---

## Recursos y Herramientas de Referencia

**Aprendizaje:** StaySafeOnline.org · Krebs on Security · Electronic Frontier Foundation (EFF) · privacytools.io

|Categoría|Herramientas|
|---|---|
|Contraseñas|Bitwarden, 1Password|
|2FA|Google Authenticator, Authy|
|VPN|ProtonVPN, NordVPN|
|Navegador|Firefox, Brave|
|Correo|ProtonMail, Tutanota|
|Mensajería|Signal, Telegram|