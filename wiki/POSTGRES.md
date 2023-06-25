## ПРЕДИСЛОВИЕ

Реализовал развертывание базы данных `PostgresSQL 15.3` и веб интерфейса `PgAdmin 4` на сервере `VK Cloud` с операционной системной `Linux OpenSUSE LEAP 15.4`.

Развертывание программного обеспечения производилось при помощи `Docker`, это позволило достаточно быстро и без проблем с зависимостями поднять базу данных и интерфейс `PgAdmin 4`. Также было решено мною создать собственный `SSL сертификат` для обеспечения защищенной работы с `PgAdmin 4` по HTTPS. Сертификат был создан самописный, потому что домен платный и его регистрация довольно непростая. Но при возможности можно поставить `SSL сертификат`, предоставленный специальным доверенным центром. (думаю вряд ли нужно запариваться над этим)

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

-  `PgAdmin` веб-сайт был поднят следующим образом:
1.   Создание томов для докер контейнера
```
mkdir ~/home/docker_volumes/
mkdir ~/home/docker_volumes/certs/
mkdir ~/home/docker_volumes/pgadmin/
```
2. Создание SSL сертификата
```
sudo zypper install openssl
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
> В случае, если файлы для сертификата были созданы не в папке /docker_volumes/certs
```
cp /path/to/server.crt  ~/home/docker_volumes/certs/
cp /path/to/server.key  ~/home/docker_volumes/certs/
```
3. Изменяем владельца обоих созданных директорий для того, чтобы докер мог получить к ним доступ
```
sudo chown -R 5050:5050 ~/home/docker_volumes/certs/
sudo chown -R 5050:5050 ~/home/docker_volumes/pgadmin/
```
4. Подтягиваем образ `PgAdmin 4`
```
docker pull dpage/pgadmin4:latest
```
5. Создаем контейнер на основе образа
```
docker run -p 5050:443 \
      -v ~/home/docker_volumes/pgadmin:/var/lib/pgadmin \
      -v ~/home/docker_volumes/certs/server.crt:/certs/server.cert \
      -v ~/home/docker_volumes/certs/server.key:/certs/server.key \
      -v /tmp/servers.json:/pgadmin4/servers.json \
      -e 'PGADMIN_DEFAULT_EMAIL=northgate@mail.ru' \
      -e 'PGADMIN_DEFAULT_PASSWORD=<password>' \
      -e 'PGADMIN_ENABLE_TLS=True' \
      -d dpage/pgadmin4
```

- `PostgresSQL` база данных была поднята следующим образом:
1. Подтягиваем образ `PostgresSQL`
```
docker pull postgres:latest
```
2. Создаем контейнер на основе образа
```
docker run -itd \
      -e POSTGRES_USER=<username> \
      -e POSTGRES_PASSWORD=<password> \
      -p 5432:5432 \
      -v /data:/var/lib/postgresql/data \
      --name postgresql postgres
```

## PGADMIN и БАЗА ДАННЫХ
Основная база данных: `northgate`
Основная схема: `northgate`
Каждому пользователю выдал все права как на `PgAdmin`, так и на базу данных и схему.

## ДОСТУПЫ

> Ключ для подключения к ВМ также, как и пароли отправлю куда нибудь в другое место

Доступ к `PgAdmin4` можно получить по следующему адресу: https://89.208.199.85:5050/

Подключением к ВМ:
1. ssh -i <путь к ключу> (если на Linux, то не забудьте выдать права `chmod 400 <путь к ключу>`) `opensuse@89.208.199.85`
2. sudo bash

Также в базе данных `PostgresSQL` создал каждому члену команды своего пользователя, `никнейм` такой же, как и на `github`. Помимо этого были созданы для каждого человека аккаунт на `PgAdmin 4`, чтобы заходить и подключаться к серверу, никнейм такой же, как и на `github`, пароли озвучу в других местах.
