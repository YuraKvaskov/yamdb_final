# CI и CD проекта api_yamdb
<image src='https://github.com/YuraKvaskov/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg'>

## Проект включает в себя 4 шага:
- Tests: Проверка кода на соответствие стандарту PEP8 (с помощью пакета flake8) и запуск pytest. Дальнейшие шаги выполнятся только если push был в ветку master или main.
- Build_and_push_to_docker_hub: Сборка и доставка докер-образов на Docker Hub
- Deploy: автоматический деплой на боевой сервер при пуше в главную ветку main
- Send_message: отправление сообщения в Telegram

# Как запустить проект локально:

Клонируйте репозиторий и перейдите в него в командной строке:
```
git clone git@github.com:YuraKvaskov/yamdb_final.git
```
Создайте виртуальное окружение и активируйте его: python -m venv venv
```
Windows: source venv\scripts\activate
Linux/Mac: source venv/bin/activate
```
Обновите pip и установите зависимости:
```
python -m pip install -r requirements.txt  
python -m pip install --upgrade pip
```
Для запуска остальных команд перейдите в каталог api_yamdb
```
cd api_yamdb
```
Запустите миграции
```
python manage.py migrate 
```
Загрузите тестовые данные из csv файлов
```
python manage.py parser_csv
```
Создайте суперпользователя
```
python manage.py createsuperuser
```
Запустите сервер
```
python manage.py runserver  
```
Проект запущен и доступен по адресу localhost:8000

Документация API YaMDb доступна по адресу http://127.0.0.1:8000/redoc/

# Как развернуть проект на сервере:

Установите соединение с сервером:
```
ssh username@server_address
```
Проверьте статус nginx:
```
sudo service nginx status
```
Если nginx запущен, остановите его:
```
sudo systemctl stop nginx
```
Установите Docker и Docker-compose:
```
sudo apt install docker.io
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
Проверьте корректность установки Docker-compose:
```
sudo  docker-compose --version
```
Создайте папку nginx:
```
mkdir nginx
```
После деплоя:
```
Соберите статические файлы (статику):
```
sudo docker-compose exec web python manage.py collectstatic --no-input
```
Примените миграции:
```
(опционально) sudo docker-compose exec web python manage.py makemigrations
sudo docker-compose exec web python manage.py migrate --noinput
```
Создайте суперпользователя:
```
sudo docker-compose exec web python manage.py createsuperuser
```
При необходимости наполните базу тестовыми данными из ../yamdb_final/infra/:
```
sudo docker exec -i infra_web_1 python manage.py loaddata --format=json - < fixtures.json
или
sudo docker-compose exec web python manage.py loaddata fixtures.json
```
Чтобы выполнить вход в контейнер:
```
sudo docker exec -it <CONTAINER_ID> bash
```
Внутри контейнера выполните миграции:
```
python manage.py migrate
```
При необходимости наполните базу данных начальными тестовыми данными:
```
python3 manage.py shell
>>> from django.contrib.contenttypes.models import ContentType
>>> ContentType.objects.all().delete()
>>> quit()
python manage.py loaddata infra/fixtures.json

