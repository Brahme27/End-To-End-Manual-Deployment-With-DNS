# Expense Application — Manual Deployment Guide

Step-by-step instructions for manually deploying the **Expense** application on RHEL-based Linux servers. The application consists of three tiers, each covered in its own notebook.

## Architecture

| Tier | Technology | Notebook |
|------|-----------|----------|
| Frontend | Nginx (static HTML/CSS/JS) | `1-frontend-server.ipynb` |
| Backend | Node.js 20 | `2-backend-server.ipynb` |
| Database | MySQL | `3-database-server.ipynb` |

## Deployment Order

1. **Database Server** — Install and configure MySQL, set the root password.
2. **Backend Server** — Install Node.js 20, deploy the app under `/app`, create a systemd service, and load the schema into MySQL.
3. **Frontend Server** — Install Nginx, deploy the static build, and configure a reverse proxy to forward `/api/` requests to the backend.

## Prerequisites

- Three RHEL-based Linux servers (or a single server for testing).
- Root / sudo access on each server.
- Network connectivity between all three tiers.

## Quick Reference

### Database
```bash
dnf install mysql-server -y
systemctl enable mysqld
systemctl start mysqld
mysql_secure_installation --set-root-pass ExpenseApp@1
```

### Backend
```bash
dnf module disable nodejs -y
dnf module enable nodejs:20 -y
dnf install nodejs -y
useradd expense
mkdir /app
curl -o /tmp/backend.zip https://expense-builds.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
cd /app && unzip /tmp/backend.zip && npm install
# Create /etc/systemd/system/backend.service, then:
systemctl daemon-reload
systemctl start backend
systemctl enable backend
mysql -h <MYSQL-SERVER-IP> -uroot -pExpenseApp@1 < /app/schema/backend.sql
systemctl restart backend
```

### Frontend
```bash
dnf install nginx -y
systemctl enable nginx
systemctl start nginx
rm -rf /usr/share/nginx/html/*
curl -o /tmp/frontend.zip https://expense-builds.s3.us-east-1.amazonaws.com/expense-frontend-v2.zip
cd /usr/share/nginx/html && unzip /tmp/frontend.zip
# Create /etc/nginx/default.d/expense.conf with reverse proxy to backend
systemctl restart nginx
```

## credits
### 110% to Shiva Kumar Reddy(JoinDevOps)