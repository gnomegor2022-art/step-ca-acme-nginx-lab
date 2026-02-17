# Лабораторный проект: Собственный ACME CA + Certbot + Nginx

Практический DevOps-проект по развертыванию собственной инфраструктуры TLS-сертификатов в изолированной сети с использованием:

- Smallstep step-ca (собственный центр сертификации)
- ACME протокол
- Certbot
- Nginx
- Локальная DNS-резолюция через /etc/hosts

---

## Цель проекта

Развернуть собственный ACME-центр сертификации и:

- Настроить ACME provisioner
- Выпустить TLS-сертификат через Certbot
- Интегрировать сертификат в Nginx
- Настроить доверие к Root CA
- Проверить работу HTTPS
- Разобраться в типичных ошибках SSL / ACME

Проект выполнен в рамках самостоятельного изучения DevOps, PKI и TLS.

---

## Архитектура лаборатории

### Сервер CA

- Hostname: ca.local
- IP: 192.168.64.10
- ПО: Smallstep step-ca
- Порт: 443

### Web сервер

- Hostname: myhost.local
- IP: 192.168.64.20
- ПО: Nginx
- Certbot в standalone режиме

---

## Схема взаимодействия

```Клиент (curl / браузер)
        |
        | HTTPS
        v
NGINX (myhost.local)
        |
        | ACME challenge
        v
step-ca (ca.local)
```

---

## Шаг 1 — Настройка DNS через /etc/hosts

На обоих серверах:

```bash
sudo nano /etc/hosts
```

Добавить:

```192.168.64.10   ca.local
192.168.64.20   myhost.local
```

Проверка:

```getent hosts ca.local
getent hosts myhost.local
ping -c1 ca.local
ping -c1 myhost.local
```
---- 

## Шаг 2 — Настройка step-ca

Проверка сервиса:

```sudo systemctl status step-ca --no-pager
```

Добавление ACME provisioner:

```sudo step ca provisioner add acme \
  --type ACME \
  --ca-config /root/.step/config/ca.json

sudo systemctl restart step-ca
```

Проверка ACME endpoint:

```curl -k https://ca.local/acme/acme/directory
```

Если возвращается JSON — ACME работает корректно.

---

## Шаг 3 — Добавление Root CA в доверенные

На web сервере:

```curl https://ca.local:443/roots.pem -o /tmp/roots.pem

sudo cp /tmp/roots.pem /usr/local/share/ca-certificates/stepca-root.crt
sudo update-ca-certificates
```

Проверка:

```curl https://ca.local/acme/acme/directory
```

Если нет ошибки SSL — доверие к Root CA настроено.

---

## Шаг 4 — Выпуск сертификата через Certbot

Важно: Certbot в standalone режиме использует порт 80.

Остановить nginx:

```sudo systemctl stop nginx
```

Выпустить сертификат:

```sudo certbot certonly \
  --server https://ca.local/acme/acme/directory \
  --standalone \
  -d myhost.local \
  --register-unsafely-without-email \
  --agree-tos
```

Сертификаты сохраняются в:

/etc/letsencrypt/live/myhost.local/

---

## Шаг 5 — Настройка Nginx

Файл конфигурации:

/etc/nginx/sites-available/default

Пример конфигурации:

```server {
    listen 80;
    server_name myhost.local;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name myhost.local;

    ssl_certificate /etc/letsencrypt/live/myhost.local/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myhost.local/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## Шаг 6 — Проверка конфигурации:

```sudo nginx -t
sudo systemctl start nginx
```

Проверка HTTPS

```curl https://myhost.local
```

Если возвращается HTML-страница Nginx — HTTPS настроен корректно.


