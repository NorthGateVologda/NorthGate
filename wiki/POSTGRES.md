## ПРЕДИСЛОВИЕ

Реализация развертывания базы данных `PostgresSQL 15.3` и веб интерфейса `PgAdmin 4` происходила на сервере `VK Cloud` с операционной системной `Linux OpenSUSE LEAP 15.4`.

Развертывание программного обеспечения производилось при помощи `Docker`, это позволило достаточно быстро и без проблем с зависимостями поднять базу данных и интерфейс `PgAdmin 4`. Также было решено создать собственный `SSL сертификат` для обеспечения защищенной работы с `PgAdmin 4` по HTTPS. Сертификат был создан самописный, потому что домен платный и его регистрация довольно непростая. Но при возможности можно поставить `SSL сертификат`, предоставленный специальным доверенным центром.

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
Каждому пользователю выданы все права как на `PgAdmin`, так и на базу данных и схему.

**Для того, чтобы создать подключение к серверу в PgAdmin необходимо выполнить следующие действия:**
1. Нажать правой кнопкой по "Server" -> "Register" -> "Server"
<p align="center">
  <img src="https://github.com/NorthGateVologda/NorthGate/assets/72744219/783556e2-ff5e-4b35-9100-fa9b83a337a6" />
</p>

2. Во вкладке "General" прописываем имя "Server"
<p align="center">
  <img src="https://github.com/NorthGateVologda/NorthGate/assets/72744219/f06c3c9f-12af-4582-8080-e01bf2af98a9" />
</p>

4. Во вкладке "Connection" прописываем следующее:
- Host (89.208.199.85)
- Port (5432)
- Maintenance database (northgate)
- Username (Выданный вам username)
- Password (Выданный вам пароль)
<p align="center">
  <img src="https://github.com/NorthGateVologda/NorthGate/assets/72744219/74d7f7aa-f389-4632-bd19-14b5ce906a40" />
</p>

4. Нажимаем кнопку "Save".

## ДОСТУПЫ

> Ключ для подключения к ВМ также, как и пароли отправлю куда нибудь в другое место

Доступ к `PgAdmin4` можно получить по следующему адресу: https://89.208.199.85:5050/

Подключением к ВМ:
1. ssh -i <путь к ключу> (если на Linux, то не забудьте выдать права `chmod 400 <путь к ключу>`) `opensuse@89.208.199.85`
2. sudo bash
