# 🔐 SSL Certificate Expiry Alert – Kubernetes CronJob

This project provides a **Kubernetes CronJob** that periodically checks SSL/TLS certificate expiry dates in your cluster and sends alerts via **Email** and/or **Microsoft Teams** when certificates are about to expire.

---

## 📌 Features
- Runs as a Kubernetes **CronJob** (`ssl-cert-expiry-alert`).
- Configurable **alert channels**:
  - ✅ Email (via SMTP)
  - ✅ Microsoft Teams (via webhook)
- Uses Kubernetes RBAC **ServiceAccount** to list secrets/certificates.
- Configurable via **ConfigMap** and **Secret**.
- Supports **External Secrets** for Teams webhook integration (Vault/Secret Manager).

---

## 🗂️ Repository Structure

```bash
.
├── alert_config.yaml                   # ConfigMap for alert configurations
├── alert_cronjob.yaml                  # CronJob definition
├── alert_script.yaml                   # Python script ConfigMap
├── role_sa.yaml                        # ServiceAccount, Role & RoleBinding
├── teams-webhook-external-secret.yaml  # Optional: ExternalSecret for Teams webhook
└── README.md                           # Documentation (this file)
```

##  How It Works

##  CronJob Schedule

Runs daily at midnight (0 0 * * *).

Executes a Python script inside a lightweight container.

##  Certificate Check

Script (cert_alert.py) uses cryptography to check expiry dates.

##  Alerting

Sends notifications if certificates are close to expiry.

Supported channels:

📧 Email → via SMTP

💬 Teams → via webhook

##  Security

Sensitive data in Kubernetes Secrets (SMTP_PASSWORD, TEAMS_WEBHOOK).

Role/ServiceAccount provides limited RBAC read access.


## 🚀 Deployment Steps

### 1️⃣ Clone the Repository
```bash
git clone https://github.com/<your-org>/<repo-name>.git
cd <repo-name>
```
## Update alert_config.yaml
```bash
SMTP_HOST: smtp.yourdomain.com
SMTP_PORT: "587"
SMTP_USER: <smtp_user>
SMTP_FROM_EMAIL: alerts@yourdomain.com
SMTP_TO_EMAILS: user1@yourdomain.com,user2@yourdomain.com
SMTP_USE_TLS: "true"
ALERT_ON_EMAIL: "True"
ALERT_ON_TEAMS: "True"
ENVIRONMENT: "Production Main"
```

## 🔒 Create Required Secrets

SMTP Password Secret
```bash
kubectl create secret generic smtp-secure-connection \
  --from-literal=password="<your-smtp-password>" \
  -n prod
```
Teams Webhook Secret (manual method)
```bash
kubectl create secret generic certexpiry-teams-webhook \
  --from-literal=webhook="<your-teams-webhook-url>" \
  -n prod
```
(Optional) If using Vault/ExternalSecrets, update and apply:
```bash
kubectl apply -f teams-webhook-external-secret.yaml -n prod
```

## ⚡ Script ConfigMap

Ensure your Python script cert_alert.py is present inside alert_script.yaml.

## 🛡️ RBAC Setup

Apply role_sa.yaml to configure the ServiceAccount and Role.

3️⃣ Apply the Kubernetes Resources
```bash
kubectl apply -f alert_config.yaml -n prod
kubectl apply -f alert_script.yaml -n prod
kubectl apply -f role_sa.yaml -n prod
kubectl apply -f alert_cronjob.yaml -n prod
# Optional (if using ExternalSecrets for Teams)
kubectl apply -f teams-webhook-external-secret.yaml -n prod
```
