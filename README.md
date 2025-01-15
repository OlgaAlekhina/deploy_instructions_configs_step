# 1. Архитектура проекта

нарисовать схему

# 2. Установка системного ПО

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

# 3. Создание базы данных для хранения конфигов

### Открыть оболочку Postgres
```
sudo -u postgres psql
```
### Создать пользователя для Postgres
```
CREATE USER <username> WITH PASSWORD '<password>';
```
### Дать пользователю права на создание баз данных
```
ALTER USER <username> CREATEDB;
```
### Создать базу данных
```
CREATE DATABASE configs OWNER <username>;
```
# 4. Деплой сервиса configs_registry (зависит от базы данных configs)

### Создать папку для сервиса (название может быть любым) и далее работать только в ней
```
mkdir configs_registry
```
### В папке configs_registry создать файл .env с переменными окружения
```
touch .env
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
touch docker-compose.yml
```
```
version: '3.8'

services:
  api:
    image:  docker.infra.cloveri.com/cloveri.start/step/configs_registry/reg_prod:latest
    restart: unless-stopped
    network_mode: "host"
    env_file:
      - ./.env
    volumes:
      - ./entrypoint.sh:/entrypoint.sh
```
### В папке configs_registry создать файл entrypoint.sh для запуска Gunicorn в контейнере (в файле можно поменять порт, на котором будет работать сервис)
```
touch entrypoint.sh
```
```
#!/bin/bash

gunicorn -b 0.0.0.0:8002 --workers 2 registry_factory.wsgi
```
### Изменить права на файл entrypoint.sh
```
chmod +x entrypoint.sh
```
### Войти в Gitlab Container Registry со своими учетными данными
```
docker login docker.infra.cloveri.com
```
### Запустить контейнер 
```
docker-compose up -d
```
### Узнать ID контейнера
```
docker ps
```
### Войти в контейнер
```
docker exec -t -i <container_id> bash
```
### Сделать миграции
```
python manage.py migrate
```
### Выйти из контейнера
```
exit
```

# 5. Деплой сервиса configs_service (зависит от сервиса configs_registry)

### Создать папку для сервиса (название может быть любым) и далее работать только в ней
```
mkdir configs_service
```
### В папке configs_service создать файл .env с переменными окружения
```
touch .env
```
| Переменная          | Пример значения  | Описание                              |
| ------------------- | -----------------| ------------------------------------- |
| SECRET_KEY          | some_key         | Секретный ключ Django - случайно сгенерированная строка                      |
| DEBUG               | True             | True или пустая строка (= False) |
| ALLOWED_HOSTS       | 127.0.0.1,cfg.step.skroy.ru        | Список allowed hosts (через запятую без пробела)          |
| REGISTRY_URL        | http://127.0.0.1          | URL, на котором работает configs_registry                         |
| REGISTRY_PORT       | 8002             | Порт, на котором работает configs_registry                         |
| ACCESS_TOKEN_PUBLIC_KEY         | public_key    | Публичный ключ для расшифровки JWT-токенов, полученных в Центре пользователей                    |
| JWT_ALGORITHM             | RS256        | Алгоритм шифрования JWT-токенов в Центре пользователей               |

### В папке configs_service создать файл docker-compose.yml с конфигурациями докера
```
touch docker-compose.yml
```
```
version: '3.8'

services:
  api:
    image:  docker.infra.cloveri.com/cloveri.start/step/configs_service/cfs_prod:latest
    restart: unless-stopped
    network_mode: "host"
    env_file:
      - ./.env
    volumes:
      - ./entrypoint.sh:/entrypoint.sh
```
Примечание: 
network_mode: "host" нужен, если взаимодействие с configs_registry происходит через localhost, если по доменному имени - лучше поменять на порты

### В папке configs_service создать файл entrypoint.sh для запуска Gunicorn в контейнере (в файле можно поменять порт, на котором будет работать сервис)
```
touch entrypoint.sh
```
```
#!/bin/bash

gunicorn -b 0.0.0.0:8001 --workers 2 configs_service.wsgi
```
### Изменить права на файл entrypoint.sh
```
chmod +x entrypoint.sh
```
### Войти в Gitlab Container Registry со своими учетными данными
```
docker login docker.infra.cloveri.com
```
### Запустить контейнер 
```
docker-compose up -d
```

# 6. Деплой сервиса step (зависит от сервиса configs_service)

### Создать папку для сервиса (название может быть любым) и далее работать только в ней
```
mkdir step_backend
```
### В папке step_backend создать файл .env с переменными окружения
```
touch .env
```
| Переменная          | Пример значения  | Описание                                                                      |
| ------------------- | -----------------|-------------------------------------------------------------------------------|
| SECRET_KEY          | some_key         | Секретный ключ Django - случайно сгенерированная строка                       |
| DEBUG               | 1                | Значение "1" соответсвует True, "0" или пустая строка - False                 |
| DJANGO_ALLOWED_HOSTS       | api.step.skroy.ru 127.0.0.1 step.skroy.ru        | Список allowed hosts (через пробел)                                           |
| DJANGO_CORS_ALLOWED_ORIGINS             | http://127.0.0.1 https://step.skroy.ru          | Список allowed origins (через пробел)                                         |
| BASE_URL             | https://api.beta.raida-dev.ru     | URL для запросов в API Raida                                                  |
| USER_RAIDA         | raida@raida.com    | Имя пользователя для запросов в API Raida                                     |
| PASSWD_RAIDA             | raida        | Пароль пользователя для запросов в API Raida                                  |
| ACCESS_TOKEN_PUBLIC_KEY         | public_key    | Публичный ключ для расшифровки JWT-токенов, полученных в Центре пользователей |
| JWT_ALGORITHM             | RS256        | Алгоритм шифрования JWT-токенов в Центре пользователей                        |
| CONFIGS_SERVICE_URL             | https://cfg.step.skroy.ru/        | URL, на котором работает configs_service                                      |
| LANGUAGE_CODE         | ru    | Основной язык сервиса                                                         |
| TIME_ZONE             | Europe/Moscow        | Часовой пояс сервиса                                                          |

### В папке step_backend создать файл docker-compose.yml с конфигурациями докера (в файле можно поменять порты, на которых работает сервис)
```
touch docker-compose.yml
```
```
version: '3.8'

services:
  api:
    image:  docker.infra.cloveri.com/cloveri.start/step/step_latest/step_prod:latest
    restart: unless-stopped
    ports:
      - 8080:8080
    env_file:
      - ./.env
    volumes:
      - ./entrypoint.sh:/entrypoint.sh
```
Примечание: 
Если взаимодействие с configs_service происходит через localhost, порты надо убрать и вместо них вставить network_mode: "host"

### В папке step_backend создать файл entrypoint.sh для запуска Gunicorn в контейнере (в файле можно поменять порт, на котором будет работать сервис)
```
touch entrypoint.sh
```
```
#!/bin/bash

gunicorn -b 0.0.0.0:8080 --workers 2 step.wsgi
```
### Изменить права на файл entrypoint.sh
```
chmod +x entrypoint.sh
```
### Войти в Gitlab Container Registry со своими учетными данными
```
docker login docker.infra.cloveri.com
```
### Запустить контейнер 
```
docker-compose up -d
```
# 7. Настройка веб-сервера Nginx

Для каждого сервиса, доступного извне по доменному имени, настраивается Nginx в качестве обратного прокси. Для этого в блок server надо вставить:

```
location / {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
        }
```
Где proxy_pass - локальный адрес, на котором запущен сервис.

На тестовом сервере использовались следующие основные настройки:

Для configs_service

```
server {
        listen 8000;

        server_name .cfg.step.skroy.ru;

        location / {
            proxy_pass http://127.0.0.1:8001;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
        }

}
```

Для step

```
server {
        listen 8000;

        server_name .step.skroy.ru;

        location / {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
        }
}
```
Запросы к configs_registry осуществлялись только по localhost без участия Nginx.

# 8. Добавление конфигов в реестр

Работа всех методов в апи step зависит от наличия в реестре нужных конфигов в правильном формате. В случае использования тестового API Raida набор конфигов должен быть следующим:
```
"task_status_id": {
      "new": "e9672f87-586d-41b1-b9b5-79acfaf6f9c2",
      "approved": "84d69c46-af76-44f3-acfd-213afc3ab1cf",
      "completed": "b9d7d6d4-e22e-4aa1-bf2d-35cdfcb94a2d",
      "rejection": "678228a5-cade-451e-bd76-33df0e0875e9"
    },
    "node_id": {
      "value": "4fc9986b-d03b-4801-a672-a191c941e17c"
    },
    "contest_status_id": {
      "new": "fe5c453a-0249-4b22-980e-c66a76ad78a9",
      "done": "6c419c06-06fd-43c3-85ee-2f58110db712",
      "voting": "a1b783c5-2dad-4338-98a8-0a6dfdb74f02",
      "no_winner": "2267301e-6b6b-4f67-9e8d-65d2c6c9c05e",
      "rejection": "678228a5-cade-451e-bd76-33df0e0875e9",
      "sum_results": "ab7d8f83-386e-4613-a062-99162356e7ad",
      "acceptance_works": "090e4125-f716-4427-a0a2-3e8e4655ae4d",
      "acceptance_works_done": "0db1196c-c03f-4d5b-a422-f15a94f7dba4"
    },
    "task_process_id": {
      "value": "eb63f559-62b1-4666-94a4-2ecdc928bdef"
    },
    "contest_process_id": {
      "value": "9e57ac56-9ce8-43fe-a725-d6eb6cb3758b"
    }
  ```
Для других проектов значения могут меняться, но структура данных и названия оставаться прежними, иначе сервис будет выдавать ошибку:

{"detail":{"code":"SERVICE_ERROR","message":"Неправильный формат конфигураций для проекта"}}

