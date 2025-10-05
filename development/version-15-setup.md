
# 🧩 Frappe + ERPNext + HRMS + CRM + Raven + NextAI (Development Setup)

This document provides step-by-step instructions to set up a full **Frappe Framework** development environment using **Frappe Docker**, including installation of **ERPNext**, **HRMS**, **CRM**, **Raven**, and **NextAI** apps.

---

## 📘 Prerequisites

Make sure you have the following installed:

* **Docker** and **Docker Compose**
* **Git**
* **VS Code** (with **Dev Containers** extension)
* **nvm (Node Version Manager)**
* **pyenv** (Python version manager)

---

## ⚙️ 1. Clone the Repository

```bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

---

## 🧱 2. Setup VSCode Development Environment

Copy the VSCode dev container examples:

```bash
cp -R devcontainer-example .devcontainer
cp -R development/vscode-example development/.vscode
```

---

## 🧩 3. Reopen in Container

Open this project in **VS Code**, and when prompted, click

> **“Reopen in Container”**

This will start the Frappe Docker Dev Container environment.

---

## 🐍 4. Initialize Bench

Inside the container terminal, run:

```bash
nvm use v18
PYENV_VERSION=3.10.13 bench init --skip-redis-config-generation --frappe-branch version-15 frappe-bench
cd frappe-bench
```

---

## 🧰 5. Configure Bench Connections

Set the global configurations for MariaDB and Redis:

```bash
bench set-config -g db_host mariadb
bench set-config -g redis_cache redis://redis-cache:6379
bench set-config -g redis_queue redis://redis-queue:6379
bench set-config -g redis_socketio redis://redis-queue:6379
```

---

## 🏗️ 6. Create a New Site

```bash
bench new-site --db-root-password 123 --admin-password admin --mariadb-user-host-login-scope=% development.localhost
```

Set the created site as the current site:

```bash
bench use development.localhost
```

Enable developer mode and clear cache:

```bash
bench --site development.localhost set-config developer_mode 1
bench --site development.localhost clear-cache
```

---

## 📦 7. Install Frappe Apps

### 🧾 ERPNext

```bash
bench get-app --branch version-15 --resolve-deps erpnext
bench --site development.localhost install-app erpnext
```

### 👨‍💼 HRMS

```bash
bench get-app --branch version-15 hrms
bench --site development.localhost install-app hrms
```

### 📊 CRM

```bash
bench get-app --branch main crm
bench --site development.localhost install-app crm
```

### 💬 Raven

```bash
bench get-app --branch main https://github.com/The-Commit-Company/raven
bench --site development.localhost install-app raven
```

### 🤖 NextAI

```bash
bench get-app --branch version-15 https://github.com/erpnextai/next_ai.git
bench --site development.localhost install-app next_ai
```

---

## 🧹 8. Other Useful Commands

### 🔄 Rebuild or Restart

```bash
bench restart
```

### 🗑️ Drop Site

```bash
bench drop-site development.localhost
```

### ❌ Uninstall an App

```bash
bench --site development.localhost uninstall-app erpnext
```

### 🔧 Remove App Completely

```bash
bench remove-app raven
```

---

## ✅ Setup Complete!

You now have a fully functional **Frappe development environment** running inside Docker with:

* **ERPNext**
* **HRMS**
* **CRM**
* **Raven**
* **NextAI**

Access your site via:

🌐 **[http://development.localhost:8000](http://development.localhost:8000)**

