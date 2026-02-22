# Como configurar SSL (HTTPS)

### 🔁 Fluxo final

```
Browser (HTTPS)
   ↓
Nginx (SSL / HTTPS)
   ├── /        → React (build estático)
   └── /api     → Node.js (HTTP interno)
```

👉 **O SSL fica somente no Nginx**
👉 **Node.js não precisa lidar com HTTPS**

---

# 1️⃣ Conceitos importantes (antes de começar)

### 📌 O que é SSL/TLS

* Garante **criptografia**
* Garante **integridade**
* Garante **autenticidade** (cadeado no navegador)

### 📌 Por que SSL no Nginx?

* Melhor performance
* Configuração centralizada
* Node fica mais simples
* Escalável

> ✅ **Boa prática universal**: *SSL termina no Nginx*

---

# 2️⃣ Pré-requisitos

✔ Servidor Linux
✔ Nginx instalado
✔ Aplicação **React buildada**
✔ Aplicação **Node.js rodando (ex: porta 3000)**
✔ Domínio (ex: `meusite.com`)

> ⚠️ SSL **não funciona corretamente apenas com IP** (para browsers)

---

# 3️⃣ Estrutura final esperada

```text
/var/www/frontend/        → React build
    ├── index.html
    ├── assets/
    └── ...

Node.js
    └── http://localhost:3000
```

---

# 4️⃣ Obtendo certificado SSL (Let’s Encrypt)

### 📦 Instalar Certbot

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

(CentOS)

```bash
sudo yum install certbot python3-certbot-nginx
```

---

### 🔐 Gerar certificado

```bash
sudo certbot --nginx -d meusite.com -d www.meusite.com
```

✔ Certificado gerado
✔ Renovação automática configurada
✔ Nginx atualizado automaticamente

---

# 5️⃣ Configurando o Nginx para React + Node + SSL

## 🧩 Arquivo de configuração

```bash
sudo nano /etc/nginx/conf.d/meusite.conf
```

---

## 🔁 Redirecionar HTTP → HTTPS

```nginx
server {
    listen 80;
    server_name meusite.com www.meusite.com;

    return 301 https://$host$request_uri;
}
```

👉 Garante que **tudo use HTTPS**

---

## 🔐 Servidor HTTPS principal

```nginx
server {
    listen 443 ssl http2;
    server_name meusite.com www.meusite.com;

    # SSL (gerenciado pelo Certbot)
    ssl_certificate /etc/letsencrypt/live/meusite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/meusite.com/privkey.pem;

    # Segurança básica
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    # =========================
    # FRONTEND REACT
    # =========================
    root /var/www/frontend;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    # =========================
    # BACKEND NODE
    # =========================
    location /api {
        proxy_pass http://localhost:3000;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

---

# 6️⃣ Backend Node.js (configuração correta)

### 📌 O Node NÃO precisa de HTTPS

```js
const express = require('express');
const app = express();

app.get('/api/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(3000, () => {
  console.log('API rodando na porta 3000');
});
```

👉 HTTPS já foi tratado no Nginx

---

# 7️⃣ Rodando Node corretamente (produção)

```bash
npm install -g pm2
pm2 start server.js --name api
pm2 save
pm2 startup
```

✔ Restart automático
✔ Logs
✔ Alta disponibilidade

---

# 8️⃣ Testes finais

### 🔍 Testar Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 🌐 Testar no navegador

* [https://meusite.com](https://meusite.com) → React
* [https://meusite.com/api/health](https://meusite.com/api/health) → Node

✔ Cadeado verde
✔ Sem erro de Mixed Content

---

# 9️⃣ Erros comuns (atenção!)

❌ Node usando HTTPS junto com Nginx
❌ React chamando `http://` no backend
❌ Porta 3000 exposta publicamente
❌ Certificado manual sem renovação

---

# 🔒 Arquitetura recomendada (resumo)

```
Internet
   ↓ HTTPS
Nginx (SSL, cache, segurança)
   ├── React (static)
   └── Node.js (proxy interno)
```