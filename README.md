# Kittygram
Kittygram - это социальная сеть для обмена фотографиями и достижениями котиков. 
Зарегистрированные пользователи могут создавать, просматривать, редактировать и удалять записи о своих питомцах. 
В профилях котиков можно указывать имя, год рождения, окрас и достижения. 
Редактировать профиль может только его владелец. 
Проект создан с использованием Docker, CI/CD и GitHub Actions для автоматического деплоя. 
Приложение развернуто на сервере в контейнерах и обновляется автоматически при внесении изменений в основную ветку main репозитория на GitHub.

## Teхнологии
- Docker
- Postgres
- Python 
- Node.js 
- Git
- Nginx
- Gunicorn
- Django (backend)
- React (frontend)

##  Запуск проекта

### GitHub Actions 
Перейдите в настройки репозитория — Settings,
выберите на панели слева Secrets and Variables → Actions, 
нажмите New repository secret. 
Сохраните представленные ниже переменные, затем нажмите Add secret.
```
- DOCKER_PASSWORD - ваш_username_docker_hub
- DOCKER_USERNAME - ваш_username_docker_hub
- HOST - IP-адрес вашего сервера
- SSH_KEY - закрытый SSH-ключ для вашего сервера
- SSH_PASSPHRASE - пароль от вашего сервера
- USER - имя пользователя от вашего сервера
- TELEGRAM_TO - ID вашего телеграм-аккаунта
- TELEGRAM_TOKEN - токен вашего бота (получить этот токен можно у телеграм-бота @BotFather)
```

### .env
Для корректной работы backend-части проекта, создайте в корне файл .env и заполните его переменными по примеру из файла .env.example
```
POSTGRES_DB=<БазаДанных>
POSTGRES_USER=<имя пользователя>
POSTGRES_PASSWORD=<пароль>
DB_NAME=<имя БазыДанных>
DB_HOST=db
DB_PORT=5432

SECRET_KEY=<ключ Django>
DEBUG=<DEBUG True/False>
ALLOWED_HOSTS='IP_адрес_сервера, 127.0.0.1, localhost, домен_сервера'
```

### Разворачиваем контейнеры backend, frontend, gateway, postgres 
```
docker compose -f docker-compose.production.yml up
```
Выполняем миграцию и собираем статику 
```
docker compose -f docker-compose.production.yml exec backend python manage.py migrate
```
```
docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
```
```
docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

### Cоздаём папку проекта kittygram на своем сервере и переходим в нее
```
mkdir kittygram
```
```
cd kittygram
```

### Скопируем файл .env на сервер, в директорию kittygram/
```
scp -i path_to_SSH/SSH_name .env \
    username@server_ip:/home/username/kittygram/.env
```

### На сервере в редакторе nano открываем конфиг Nginx
```
sudo nano /etc/nginx/sites-enabled/default
```
### Изменяем настройки location в секции server
```
# Kittigram server configuration
server {
    server_tokens off;
    server_name имя_вашего_сайта;

        location /media/ {
            proxy_pass http://127.0.0.1:9000;
            client_max_body_size 20M;
        }

        location /api/ {

            proxy_pass http://127.0.0.1:9000;
            client_max_body_size 20M;
        }

        location /admin/ {
            proxy_pass http://127.0.0.1:9000;
            client_max_body_size 20M;
        }

        location / {
            proxy_pass http://127.0.0.1:9000;
        }
```
## Как проверить работу с помощью автотестов

В корне репозитория создайте файл tests.yml со следующим содержимым:
```yaml
repo_owner: ваш_логин_на_гитхабе
kittygram_domain: полная ссылка (https://доменное_имя) на ваш проект Kittygram
taski_domain: полная ссылка (https://доменное_имя) на ваш проект Taski
dockerhub_username: ваш_логин_на_докерхабе
```

Скопируйте содержимое файла `.github/workflows/main.yml` в файл `kittygram_workflow.yml` в корневой директории проекта.

Для локального запуска тестов создайте виртуальное окружение, установите в него зависимости из backend/requirements.txt и запустите в корневой директории проекта `pytest`.

## Автор проекта

- [Дарья Анохина](https://github.com/Destedora)