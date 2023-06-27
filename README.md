Описание проекта
Kittygram - сервис для любителей котиков и кошечек, а также любой кошачей милоты.

Возможности проекта :

Редактировать, добавлять, удалять, просматривать котов;
Добавлять новые / присваивать существующие достижения;
Просматривать чужих котов и их достижения;

Установка
Клонируйте репозиторий:

git clone ...
cd kittygram
Создайте файл .env и заполните его своими данными:

POSTGRES_DB= [имя_базы_данных]
POSTGRES_USER=[имя_пользователя_базы]
POSTGRES_PASSWORD=[пароль_к_базе]
DB_HOST=[имя_хоста]
DB_PORT=[порт_соединения_к_базе]
Создание Docker-образов
Замените username на ваш логин на DockerHub:

cd frontend
docker build -t username/kittygram_frontend .
cd ../backend
docker build -t username/kittygram_backend .
cd ../nginx
docker build -t username/kittygram_gateway . 
Загрузите образы на DockerHub:

docker push username/kittygram_frontend
docker push username/kittygram_backend
docker push username/kittygram_gateway
Деплой на сервере
Подключитесь к удаленному серверу

ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
Создайте на сервере директорию kittygram через терминал

mkdir kittygram
Установка docker compose на сервер:

sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt-get install docker-compose-plugin
В директорию kittygram/ скопируйте файлы docker-compose.production.yml и .env:

scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
Запустите docker compose в режиме демона:

sudo docker compose -f docker-compose.production.yml up -d
Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
На сервере в редакторе nano откройте конфиг Nginx:

nano /etc/nginx/sites-enabled/default
Измените настройки location в секции server:

location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:8000;
}
Проверьте работоспособность конфига и перезапустите Nginx:

sudo nginx -t 
sudo service nginx reload
Настройка CI/CD
Файл workflow уже написан. Он находится в директории

kittygram/.github/workflows/main.yml
Для адаптации его на своем сервере добавьте секреты в GitHub Actions:

SECRET_KEY # стандартный ключ, который создается при старте проекта

DOCKER_USERNAME # имя пользователя в DockerHub DOCKER_PASSWORD # пароль пользователя в DockerHub HOST # ip_address сервера USER # имя пользователя SSH_KEY # приватный ssh-ключ (cat ~/.ssh/id_rsa) PASSPHRASE # кодовая фраза (пароль) для ssh-ключа

TELEGRAM_TO # id телеграм-аккаунта (можно узнать у @userinfobot, команда /start) TELEGRAM_TOKEN # токен бота (получить токен можно у @BotFather, /token, имя бота) ```
