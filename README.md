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
