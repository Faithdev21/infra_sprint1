# Kittygram
***
## Kittygram - проект для обмена фотографиями
***
## Технологии
* Python 3.10
* Django3
* djangorestframework
* Nginx
* Gunicorn
***
## Установка на локальный сервер
1. Клонировать проект из репозитория:  
```git clone git@github.com:Faithdev21/infra_sprint1.git```
2. Установить виртуальное окружение:  
```python3 -m venv venv```
3. Установить зависимости:  
```pip install -r requirements.txt```
4. Создать в корневой папке файл ***.env***, добавить SECRET_KEY, TIME_ZONE, USE_TZ.
***
# Деплой проекта на удаленный сервер
***
 ### Подключение к GitHub
Установить Git:  
```sudo apt install git```  
Сгенерировать пару SSH-ключей:  
```ssh-keygen```  
Сохранить открытый ключ на gitHub аккаунте:  
```cat .ssh/id_rsa.pub```  
Клонировать проект на удаленный сервер:  
```git clone git@github.com:Ваш_аккаунт/Название_проекта.git```  
***
### Запуск бэкенда
Установить на сервер пакетный менеджер и утилтиту для создания виртуального окружения:  
```sudo apt install python3-pip python3-venv -y```  
Создать виртуальное окружение:  
```python3 -m venv venv```  
Активировать виртуальное окружение:  
```source venv/bin/activate```  
Установить зависимости:  
```pip install -r requirements.txt```  
Выполнить миграции:   
```python manage.py migrate```  
Создать суперпользователя:  
```python manage.py createsuperuser```  
В файле settings.py в ALLOWED_HOSTS добавить IP-адрес сервера, а так же '127.0.0.1', 'localhost' и доменное имя.  
***
### Запуск фронтенда
Установить на сервер пакетный менеджер npm:  
```curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\```  
```sudo apt-get install -y nodejs```  
Установить зависимости для фронтенда из директории ***infra_sprint1/frontend***:  
```npm i```  
***
### Установка и запуск Gunicorn
```pip install gunicorn==20.1.0```  
Создать юнит для gunicorn:  
```sudo nano /etc/systemd/system/gunicorn.service ```  
В файле gunicorn.service описать конфигурацию процесса:
```html
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=yc-user 
WorkingDirectory=/home/yc-user/taski/backend/
ExecStart=/home/yc-user/taski/backend/venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target  
```

Запустить процесс gunicorn.service:  
```sudo systemctl start gunicorn```  
Добавим процесс в автозапуск:  
```sudo systemctl enable gunicorn```  
***
### Установка Nginx
```sudo apt install nginx -y```  
Запустить Nginx:  
```sudo systemctl start nginx```  
Указать файрволу, какие порты должны остаться открытыми:  
```sudo ufw allow 'Nginx Full'```  
```sudo ufw allow OpenSSH```  
Включить файрвол:  
```sudo ufw enable```  
***
### Собрать статику фронтенд-приложения
Из директории ***infra_sprint1/frontend*** запустить:  
```npm run build```  
```sudo cp -r /home/yc-user/infra_sprint1/frontend/build/. /var/www/kittygram/```  
Описываем настройки для работы со статикой фронтенд-приложения:   
``` sudo nano /etc/nginx/sites-enabled/default```  
Удалить все найтроки из файла и добавить:  
```html
server {
    server_name ВАШ_IP ВАШ ДОМЕН;
    client_max_body_size 32m;
    server_tokens off;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }
    location / {
        root   /var/www/kittygram;
        index  index.html index.htm;
        try_files $uri /index.html;
    }
```
***
### Описать настройки для работы с бэкенд-приложением
В файле ***settings.py*** изменить(добавить):  
```STATIC_URL = 'static_backend'```
```STATIC_ROOT = BASE_DIR / 'static_backend'```  
При активированном виртуальном окрежении выполните команду:  
```python manage.py collectstatic```  
Перейдите в корень проекта и выполните команду:  
```sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/```  
***
### Натройка шифрования
Установить ***certbot***:  
```sudo apt install snapd```  
```sudo snap install core; sudo snap refresh core```  
```sudo snap install --classic certbot```  
```sudo ln -s /snap/bin/certbot /usr/bin/certbot```  
Запустим certbot и получим SSL-сертификуат:  
```sudo certbot --nginx```  
Перезагрузить конфигурацию Nginx:  
```sudo systemctl reload nginx```  
Настройка автоматического обновления SSL-сертификата:  
```sudo certbot certificates```  
```sudo certbot renew --dry-run```  
### Настройка мониторинга и сбор ошибок
Зарегестрируйтесь на сайте <small>[Uptimerobot](https://uptimerobot.com/).  
После этого на гланом дашборбе нажмите зеленую кнопку ***Add New Monitor***, в открывшемся окне укажите тип монитора: HTTP(s), придумайте имя для этого монитора, введите URL, который хотите мониторить.  
Автор проекта  | GitHub
------------- | -------------
Лоскутов Егор  | <small>[GitHub](https://github.com/Faithdev21)
