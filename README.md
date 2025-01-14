# 1. Установка системного ПО

### Установить Nginx
```
sudo apt install nginx
```

### Установить Docker
```
sudo snap install curl
```
```
curl -fsSL https://get.docker.com/ | sh
```

### Добавить пользователя в группу docker
```
sudo usermod -aG docker <username>
```

### Установить Docker-compose 
```
sudo apt  install docker-compose
```

### Установить PostgreSQL
```
sudo apt install postgresql postgresql-contrib
```

# 2. Архитектура проекта

нарисовать схему

# 3. Создание базы данных для хранения конфигов

### Открыть оболочку Postgres
```
sudo -u postgres psql
```
### Создать пользователя для Postgres
```
CREATE USER <username> WITH PASSWORD <password>;
```
### Дать пользвателю права на создание баз данных
```
ALTER USER <username> CREATEDB;
```
### Создать базу данных
```
CREATE DATABASE configs;
```
# 4. Разворачивание сервиса configs_registry (зависит от базы данных configs)

### Создать папку для сервиса (название может быть любым)
```
mkdir configs_registry
```
### В папке configs_registry создать файл .env с переменными окружения
```
touch configs_service/.env
```
| Переменная          | Пример значения  | Описание                              |
| ------------------- | -----------------| ------------------------------------- |
| SECRET_KEY          | some_key         | Секретный ключ Django - случайно сгенерированная строка                      |
| DEBUG               | True             | True или пустая строка (= False) |
| ALLOWED_HOSTS       | 127.0.0.1        | Список allowed hosts (через запятую без пробела)          |
| DB_NAME             | configs          | Название базы данных                         |
| DB_USER             | configs_user     | Пользователь базы данных                         |
| DB_PASSWORD         | some_password    | Пароль пользователя базы данных                     |
| DB_HOST             | 10.0.1.16        | Адрес хоста базы данных               |
| DB_PORT             | 5432             | Порт базы данных                        |

### В папке configs_registry создать файл docker-compose.yml с конфигурациями докера
```
touch configs_service/docker-compose.yml
```
```
version: '3.8'

services:
  api:
    image:  docker.infra.cloveri.com/cloveri.start/step/configs_registry
    restart: unless-stopped
    network_mode: "host"
    env_file:
      - ./.env
    volumes:
      - ./entrypoint.sh:/entrypoint.sh
```
### В папке configs_registry создать файл entrypoint.sh для запуска Gunicorn в контейнере (в файле можно поменять порт, на котором будет работать приложение)
```
touch configs_service/entrypoint.sh
```
```
#!/bin/bash

gunicorn -b 0.0.0.0:8002 --workers 2 step.wsgi
```
### Изменить права на файл entrypoint.sh
```
chmod +x entrypoint.sh
```
### Войти в Gitlab Registry со своими учетными данными
```
docker login docker.infra.cloveri.com
```
### Запустить контейнер (только из папки сервиса)
```
docker-compose up -d
```
# 5. Разворачивание сервиса configs_service (зависит от сервиса configs_registry)

### Создать папку для сервиса (название может быть любым)
```
mkdir configs_service
```
### В папке configs_service создать файл .env с переменными окружения
```
touch configs_service/.env
```
| Переменная          | Пример значения  | Описание                              |
| ------------------- | -----------------| ------------------------------------- |
| SECRET_KEY          | some_key         | Секретный ключ Django - случайно сгенерированная строка                      |
| DEBUG               | True             | True или пустая строка (= False) |
| ALLOWED_HOSTS       | 127.0.0.1,cfg.step.skroy.ru        | Список allowed hosts (через запятую без пробела)          |
| REGISTRY_URL        | http://127.0.0.1          | URL, на котором работает configs_registry                         |
| REGISTRY_PORT       | 8002             | порт, на котором работает configs_registry                         |
| ACCESS_TOKEN_PUBLIC_KEY         | public_key    | Публичный ключ для расшифровки JWT-токенов, полученных в Центре пользователей                    |
| JWT_ALGORITHM             | RS256        | Алгоритм шифрования JWT-токенов в Центре пользователей               |

ACCESS_TOKEN_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----\nMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAwwP2xW8sTF63kyi75Esy\nJd5+ENBZifoOwFz5JTjXRP0yg/feRIR1F3EJ3NMQy7uXXuTCL09wAKBcqxjilXhS\nXLNpBFOZV2ESs3vqAwLL/xN25QWQMUzvCWwVcU3CKrIgDTtcYeyj0xnGjpO9cB8W\nBdtkloxOAXjZaqQ8WLLHtkp2bc34kp4vivFwR8o21v2oeVsINH0eb5Ci8jCDKs6U\nh4Yml6EsRAlKVMJYOsgWm3J9TkbKCvpgl5XcTYCyVdQMRlcmFyF2mG90nyo0tv13\n6oxqGPP7GKozYWIQ1wAprhWPYf13m8/Agvw5bLJknybUO77rVaVM+hq4ASXNMY+j\nyhFfO/OYPZOkfLdmu1UhbIXwy1cHWbk9F6MWPF7fgU/mNVgUlibmUh+zEqdjB/Hx\nCfQPnKFEmmsQhZiLMfcEOY15OTnm9UoM8K0xZQpCM4Hj6v/LVeQnyddeudIgAa0H\nEBH0AqEynXBiJUPDMlp17rJLQsWh03fmTq8W+t41sVk8N1MXJ8dndix7JrRYI9Zx\nMR6aNehyXLCxZfw7Hpr8J5AeMNEBMogkQo83hE0DNURcr/l09pYDu4kxhuzSc1DV\nLKFYpt7G4ZxVDjYY6v8045y5UBdge4KovZjagSmOK/rraWTRyNtPSqqrH0YIlSWi\nCn5sN6gYwyFEYl3uUiTBJScCAwEAAQ==\n-----END PUBLIC KEY-----

