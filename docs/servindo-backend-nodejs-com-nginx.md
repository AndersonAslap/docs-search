Existem **várias formas de servir (expor) um backend Node.js usando o Nginx**, e a escolha depende de **escala, segurança, simplicidade e arquitetura**.

Vou te explicar **do básico ao profissional**, com **quando usar cada abordagem**, exemplos e boas práticas.

---

## 1️⃣ Reverse Proxy (FORMA MAIS COMUM E RECOMENDADA)

### 📌 Como funciona

* O **Node.js** roda em uma porta interna (ex: `3000`)
* O **Nginx** escuta a porta pública (`80` / `443`)
* O Nginx **encaminha as requisições** para o Node

### 🔁 Fluxo

```
Browser → Nginx (80/443) → Node.js (3000)
```

### ✅ Vantagens

* Segurança (Node não fica exposto)
* SSL/TLS fica no Nginx
* Cache, compressão e rate limit
* Fácil de escalar

### 🧠 Quando usar

👉 **Sempre que for produção**

### 🧩 Exemplo de configuração

```nginx
server {
    listen 80;
    server_name api.meusite.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;

        proxy_cache_bypass $http_upgrade;
    }
}
```

### 🟢 Backend Node

```bash
node server.js
# ou
pm2 start server.js
```

---

## 2️⃣ Node.js SERVINDO DIRETO (SEM NGINX) ❌

### 📌 Como funciona

* Node escuta direto na porta `80` ou `443`

### ❌ Por que NÃO é recomendado

* Node não é otimizado como servidor HTTP
* SSL mais complexo
* Sem cache, gzip, rate limit
* Mais vulnerável

### 🧠 Quando usar

👉 Apenas **desenvolvimento local**

---

## 3️⃣ Nginx + PM2 (PRODUÇÃO PROFISSIONAL)

### 📌 Como funciona

* Nginx recebe requisições
* PM2 gerencia o Node.js (restart, logs, cluster)

### 🔁 Fluxo

```
Browser → Nginx → PM2 → Node.js
```

### ✅ Vantagens

* Alta disponibilidade
* Restart automático
* Logs organizados
* Clustering fácil

### 📦 Exemplo PM2

```bash
pm2 start server.js --name api --instances max
pm2 save
pm2 startup
```

### 🧠 Ideal para

👉 Backends REST, APIs, SaaS, produção real

---

## 4️⃣ Load Balancer (ESCALA)

### 📌 Como funciona

* Nginx distribui requisições para múltiplas instâncias Node

### 🔁 Fluxo

```
Browser → Nginx
            ├── Node 3001
            ├── Node 3002
            └── Node 3003
```

### 🧩 Exemplo

```nginx
upstream backend {
    server localhost:3001;
    server localhost:3002;
    server localhost:3003;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
    }
}
```

### 🧠 Quando usar

👉 Alta concorrência ou muitos usuários

---

## 5️⃣ SSL TERMINATION (HTTPS NO NGINX)

### 📌 Como funciona

* HTTPS termina no Nginx
* Node recebe HTTP simples

### 🔁 Fluxo

```
Browser (HTTPS) → Nginx (443) → Node (HTTP)
```

### ✅ Benefícios

* Certificados centralizados
* Node mais simples
* Melhor performance

### 🧩 Exemplo

```nginx
server {
    listen 443 ssl;
    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

---

## 6️⃣ Servir Frontend + Backend juntos

### 📌 Como funciona

* Nginx serve o React/Vue estático
* API vai para o Node

### 🔁 Fluxo

```
/        → React (build)
/api    → Node.js
```

### 🧩 Exemplo

```nginx
server {
    listen 80;

    root /var/www/frontend;
    index index.html;

    location /api {
        proxy_pass http://localhost:3000;
    }

    location / {
        try_files $uri /index.html;
    }
}
```

### 🧠 Ideal para

👉 SPA + API no mesmo servidor

---

## 📊 RESUMO RÁPIDO

| Cenário               | Melhor abordagem              |
| --------------------- | ----------------------------- |
| Desenvolvimento       | Node direto                   |
| Produção simples      | Nginx + Node                  |
| Produção profissional | Nginx + PM2                   |
| Alta escala           | Nginx Load Balancer           |
| HTTPS                 | SSL no Nginx                  |
| React + API           | Nginx serve front + proxy API |

---

## 🏆 MELHOR PRÁTICA GERAL

> **Sempre use o Nginx como porta de entrada e o Node.js rodando internamente.**
