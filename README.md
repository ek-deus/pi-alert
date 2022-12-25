# pi.alert

1. sudo docker-compose up -d
2. sudo docker-compose --env-file /path/to/.env up
3. 


#📕 Основное использование
Вам нужно будет запустить контейнер в сети хоста, например: `sudo docker run --rm --net=host jokobsk/pi.alert`
Начальное сканирование может занять до 15 минут (с 50 устройствами и MQTT). Последующие 3 и 5 минут, так что подождите столько времени, пока не будут запущены все проверки.

#Переменные среды Docker

|Переменная	|Описание| |По умолчанию|
|PORT	|Порт веб-интерфейса|	|20211|
|TZ|	|Часовой пояс для корректного отображения статистики. Найдите свой часовой пояс здесь|	|Europe/Berlin|
|HOST_USER_GID|	|Идентификатор пользователя (UID) для сопоставления пользователя в контейнере с пользователем сервера с достаточными разрешениями на чтение и запись в сопоставленных файлах|	|1000|
|HOST_USER_ID|	|Идентификатор группы пользователей (GID) для сопоставления группы пользователей в контейнере с группой пользователей сервера с достаточными разрешениями на чтение и запись сопоставленных файлов.|	|1000|

#Пути к Docker

|| |Путь|	|Описание|
|Требуется	|:/home/pi/pialert/config|	|Папка, в которой будет содержаться pialert.confфайл (подробности см. Ниже)|
|Требуется	|:/home/pi/pialert/db|	|Папка, в которой будет содержаться pialert.dbфайл|
|Необязательно|	|:/home/pi/pialert/db/setting_darkmode	Сопоставьте пустой файл с именемsetting_darkmode, если вы хотите принудительно включить темный режим при восстановлении контейнера|
|Необязательно|	|:/home/pi/pialert/front/log|	|Папка журналов полезна для отладки, если у вас возникли проблемы с настройкой контейнера|
|Необязательно|	|:/etc/pihole/pihole-FTL.db|	|Файл pihole-FTL.dbбазы данных PiHole. Требуется, если вы хотите использовать PiHole|
|Необязательно|	|:/etc/pihole/dhcp.leases	dhcp.leases| |Досье ПиХола. Требуется, если вы хотите использовать PiHole|

#Конфигурация (pialert.conf)

Загрузите файл pialert.conf отсюда.
❗ Установите SCAN_SUBNETSпеременную.
Адаптер , вероятно , будет eth0или eth1. (Запуститеiwconfig, чтобы найти имя (ы) вашего интерфейса).
Укажите сетевой фильтр (который значительно ускоряет процесс сканирования). Например, фильтр 192.168.1.0/24охватывает диапазоны IP-адресов от 192.168.1.0 до 192.168.1.255.
Примеры для одной и двух подсетей (❗ Обратите внимание на ['...', '...']формат для двух или более подсетей):
Одна подсеть: SCAN_SUBNETS = '192.168.1.0/24 --interface=eth0'
Две подсети: SCAN_SUBNETS = ['192.168.1.0/24 --interface=eth0', '192.168.1.0/24 --interface=eth1']
Задайте свои настройки отчетов.
Я рекомендую проверять наличие новых настроек каждый раз, когда вы перестраиваете контейнер.
🛑 Общие проблемы
💡 Перед созданием новой проблемы, пожалуйста, проверьте, была ли уже решена аналогичная проблема.

#Разрешения

При возникновении проблем (ошибки AJAX, невозможность записи в базу данных, пустой экран и т. Д.) Убедитесь, что разрешения установлены правильно, и проверьте журналы в разделе/home/pi/pialert/front/log.
Чтобы решить проблемы с разрешениями, вы также можете попытаться создать резервную копию БД, а затем запустить восстановление БД через раздел Обслуживание> Резервное копирование / восстановление.
Вы можете попробовать также установить владельца и группуpialert.db, выполнив следующие действия в хост-системе: docker exec pialert chown -R www-data:www-data /home/pi/pialert/db/pialert.db.
Сопоставьте с локальными идентификаторами пользователей и групп. Укажите переменные среды HOST_USER_IDиHOST_USER_GID, если необходимо.
Сопоставьте файл `pialert.db (⚠ не папку) `с :/home/pi/pialert/db/pialert.db`(подробности см. В примерах ниже)
Перезапуск / сбой контейнера

Проверьте журналы для получения подробной информации. Часто необходимая настройка для метода уведомления отсутствует.
не удалось разрешить узел

Убедитесь, что ваша SCAN_SUBNETSпеременная использует правильную маску, --interfaceкак указано в приведенных выше инструкциях.
Примеры создания Docker-compose можно найти ниже.

#📄 Примеры

```yml
version: "3"
services:
  pialert:
    container_name: pialert
    image: "jokobsk/pi.alert:latest"      
    network_mode: "host"        
    restart: unless-stopped
    volumes:
      - local/path/pialert/config:/home/pi/pialert/config
      - local/path/pialert/db:/home/pi/pialert/db
      # (optional) map an empty file with the name 'setting_darkmode' if you want to force the dark mode on container rebuilt
      - local/path/pialert/db/setting_darkmode:/home/pi/pialert/db/setting_darkmode
      # (optional) useful for debugging if you have issues setting up the container
      - local/path/logs:/home/pi/pialert/front/log
    environment:
      - TZ=Europe/Berlin      
      - HOST_USER_ID=1000
      - HOST_USER_GID=1000
      - PORT=20211
 ```     
Чтобы запустить контейнер, выполните: `sudo docker-compose up -d`

Пример 2
docker-compose.yml

```yml
version: "3"
services:
  pialert:
    container_name: pialert
    image: "jokobsk/pi.alert:latest"      
    network_mode: "host"        
    restart: unless-stopped
    volumes:
      - ${APP_DATA_LOCATION}/pialert/config:/home/pi/pialert/config
      - ${APP_DATA_LOCATION}/pialert/db/pialert.db:/home/pi/pialert/db/pialert.db
      # (optional) map an empty file with the name 'setting_darkmode' if you want to force the dark mode on container rebuilt
      - ${APP_DATA_LOCATION}/pialert/db/setting_darkmode:/home/pi/pialert/db/setting_darkmode
      # (optional) useful for debugging if you have issues setting up the container
      - ${LOGS_LOCATION}:/home/pi/pialert/front/log
    environment:
      - TZ=${TZ}      
      - HOST_USER_ID=${HOST_USER_ID}
      - HOST_USER_GID=${HOST_USER_GID}
      - PORT=${PORT}
```      
.env файл

```.env
#GLOBAL PATH VARIABLES

APP_DATA_LOCATION=/path/to/docker_appdata
APP_CONFIG_LOCATION=/path/to/docker_config
LOGS_LOCATION=/path/to/docker_logs

#ENVIRONMENT VARIABLES

TZ=Europe/Paris
HOST_USER_ID=1000
HOST_USER_GID=1000
PORT=20211

#DEVELOPMENT VARIABLES

DEV_LOCATION=/path/to/local/source/code
```

Чтобы запустить контейнер, выполните: `sudo docker-compose --env-file /path/to/.env up`

Пример 3
Любезно предоставлено pbek. Том pialert_dbиспользуется каталогом БД. Два файла конфигурации монтируются непосредственно из локальной папки на свои места в папке config. Вы можете создать резервную docker-compose.yamlкопию папки и папки docker volumes.

  ```yml
  pialert:
    image: jokobsk/pi.alert
    ports:
      - "80:20211/tcp"
    environment:
      - TZ=Europe/Vienna
    networks:
      local:
        ipv4_address: 192.168.1.2
    restart: unless-stopped
    volumes:
      - pialert_db:/home/pi/pialert/db
      - ./pialert/pialert.conf:/home/pi/pialert/config/pialert.conf
```

