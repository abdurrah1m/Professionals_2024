# WIKI
&ensp; &ensp; 6. Создайте в домашней директории пользователя ubuntu файл wiki.yml для приложения MediaWiki.  
&ensp; &ensp; &ensp; 1. Средствами docker compose должен создаваться стек контейнеров с приложением MediaWiki и базой данных.  
&ensp; &ensp; &ensp; &ensp; 1. Используйте два сервиса  
&ensp; &ensp; &ensp; &ensp; 2. Основной контейнер MediaWiki должен называться wiki и использовать образ mediawiki  
&ensp; &ensp; &ensp; &ensp; 3. Файл LocalSettings.php с корректными настройками должен находиться в домашней папке пользователя ubuntu и автоматически монтироваться в образ.  
&ensp; &ensp; &ensp; &ensp; 4. Контейнер с базой данных должен называться db и использовать образ mysql  
&ensp; &ensp; &ensp; &ensp; 5. Он должен создавать базу с названием mediawiki, доступную по стандартному порту, для пользователя wiki с паролем P@ssw0rd  
&ensp; &ensp; &ensp; &ensp; 6. База должна храниться в отдельном volume с названием dbvolume  
&ensp; &ensp; &ensp; &ensp; 7. База данных должна находиться в одной сети с приложением App2, но не должна быть доступна снаружи.  
&ensp; &ensp; &ensp; &ensp; 8. MediaWiki должна быть доступна извне через порт 80.

Накидываем инструкции:
```
nano ~/wiki.yml
```
```
version: '3'
services:
  MediaWiki:
    container_name: wiki
    image: mediawiki
    restart: always
    ports:
      - 80:80
    links:
      - database
    volumes:
      - images:/var/www/html/images
      # - ./LocalSettings.php:/var/www/html/LocalSettings.php

  database:
    container_name: db
    image: mysql
    restart: always
    environment:
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: wiki
      MYSQL_PASSWORD: P@ssw0rd
      MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
    volumes:
      - dbvolume:/var/lib/mysql

volumes:
  images:
  dbvolume:
    external: true
```

Вручную создаём volume:
```
docker volume create dbvolume
```

Запуск:
```
docker-compose -f wiki.yml up -d
```
