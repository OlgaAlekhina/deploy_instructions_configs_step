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
