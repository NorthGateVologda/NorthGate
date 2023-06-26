## ПРЕДИСЛОВИЕ

Реализация развертывания базы данных `PostgreSQL 15.3` с плагином `PostGIS` и веб интерфейса `PgAdmin4` происходила на сервере `VK Cloud` с операционной системной `Linux OpenSUSE LEAP 15.4`.

Развертывание программного обеспечения производилось при помощи `Docker` и `docker-compose`, это позволило достаточно быстро и без проблем с зависимостями поднять базу данных и интерфейс `PgAdmin4`, а также управлять томами и иметь возможность быстро переносить конфигурацию `Docker` на другие машины. Также было решено создать собственный `SSL сертификат` для обеспечения защищенной работы с `PgAdmin4` по `HTTPS`. Сертификат был создан самописный, потому что домен платный и его регистрация довольно непростая. Но при возможности можно поставить `SSL сертификат`, предоставленный специальным доверенным центром.

## ХАРАКТЕРИСТИКИ ВИРТУАЛЬНОЙ МАШИНЫ
<html>
<body>
<!--StartFragment-->

Параметры виртуальной машины | Значение
-- | --
Виртуальные CPU, шт. | 2
Объем оперативной памяти, МБ | 4096
Объём всех дисков, ГБ | 150

<!--EndFragment-->
</body>
</html>

## УСТАНОВКА И НАСТРОЙКА ПРОГРАММНОГО ОБЕСПЕЧЕНИЯ НА СЕРВЕРЕ

Перед началом запуска всех необходимых контейнеров для работы, необходимо создать папки для хранения `томов` и `построителей докера`:

1. Переходим в домашний каталог
```
cd $HOME/home
```
2. Создаём следующие папки для докера
```
mkdir ~/home/docker_volumes/
mkdir ~/home/docker_builders/
mkdir ~/home/docker_volumes/certs/
mkdir ~/home/docker_volumes/pgadmin/
```
3. Создаём SSL сертификат
```
sudo zypper install openssl
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
> В случае, если файлы для сертификата были созданы не в папке **docker_volumes/certs**
```
cp /path/to/server.crt  ~/home/docker_volumes/certs/
cp /path/to/server.key  ~/home/docker_volumes/certs/
```
4. Изменяем владельца созданных директорий для того, чтобы докер мог получить к ним доступ
```
sudo chown -R 5050:5050 ~/home/docker_volumes/certs/
sudo chown -R 5050:5050 ~/home/docker_volumes/pgadmin/
```
5. Переходим в созданную директорию `docker_builders`
> В случае, если не установлен **docker-compose**, необходимо его установить
для дальнейшей работы следующим образом **(для OpenSUSE)**:  
sudo zypper install python3-pip  
pip install docker-compose==1.29.2  
```
cd $HOME/home/docker_builders
```
6. Создаём файл `docker-compose.yml`
```
touch docker-compose.yml
```
7. Открываем его и вставляем следующее содержимое
> Предполагается, что все значения в треугольных скобка **<..>** будут заменены вами в файле **docker-compose.yml**
```
version: '3.9'
services:
  postgres:
    container_name: postgres
    image: postgis/postgis:latest
    environment:
      - POSTGRES_DB=northgate
      - POSTGRES_USER=<username>
      - POSTGRES_PASSWORD=<password>
    volumes:
      - ~/home/docker_volumes/pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - postgres
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=<email>
      - PGADMIN_DEFAULT_PASSWORD=<password>
      - PGADMIN_ENABLE_TLS=True
    volumes:
      - ~/home/docker_volumes/pgadmin:/var/lib/pgadmin
      - ~/home/docker_volumes/certs/server.crt:/certs/server.cert
      - ~/home/docker_volumes/certs/server.key:/certs/server.key
      - /tmp/servers.json:/pgadmin4/servers.json
    ports:
      - "5050:443"
    networks:
      - postgres
networks:
  postgres:
    driver: bridge
```
8. Выполняем команду для запуска контейнеров
```
docker-compose up -d
```

## PGADMIN и БАЗА ДАННЫХ
Основная база данных: `northgate`
Основная схема: `northgate`
Каждому пользователю выданы все права как на `PgAdmin4`, так и на базу данных и схему.

**Для того, чтобы создать подключение к серверу в PgAdmin необходимо выполнить следующие действия:**
1. Нажать правой кнопкой по `Server` -> `Register` -> `Server`
<p align="center">
  <img src="https://github.com/NorthGateVologda/NorthGate/assets/72744219/783556e2-ff5e-4b35-9100-fa9b83a337a6" />
</p>

2. Во вкладке `General` прописываем имя `Server`
<p align="center">
  <img src="https://github.com/NorthGateVologda/NorthGate/assets/72744219/f06c3c9f-12af-4582-8080-e01bf2af98a9" />
</p>

4. Во вкладке `Connection` прописываем следующее:
- **Host**: 89.208.199.85
- **Port**: 5432
- **Maintenance** database: northgate
- **Username**: Выданный вам username
- **Password**: Выданный вам пароль
<p align="center">
  <img src="https://github.com/NorthGateVologda/NorthGate/assets/72744219/74d7f7aa-f389-4632-bd19-14b5ce906a40" />
</p>

4. Нажимаем кнопку `Save`.

## ДОСТУПЫ

> Ключи доступа к виртуальной машине будут лежать на приватном google диске

Доступ к `PgAdmin4` можно получить по следующему адресу: https://89.208.199.85:5050/

Подключением к ВМ:
1. ssh -i <путь к ключу> (если на Linux, то не забудьте выдать права `chmod 400 <путь к ключу>`) `opensuse@89.208.199.85`
2. sudo bash
