# HelloFIRPO

&ensp; &ensp; 3. В домашней директории хоста создайте файл name.txt и запишите в него строку experts.  
&ensp; &ensp; 4. Напишите Dockerfile для приложения HelloFIRPO.  
&ensp; &ensp; &ensp; 1. В качестве базового образа используйте alpine  
&ensp; &ensp; &ensp; 2. Сделайте рабочей директорию /hello и скопируйте в неё name.txt  
&ensp; &ensp; &ensp; 3. Контейнер при запуске должен выполнять команду echo, которая выводит сообщение "Hello, FIRPO! Greetings from " и затем содержимое файла name.txt, после чего завершать свою работу.  
&ensp; &ensp; 5. Соберите образ приложения App и загрузите его в ваш Registry.  
&ensp; &ensp; &ensp; 1. Используйте номер версии 1.0 для вашего приложения  
&ensp; &ensp; &ensp; 2. Образ должен быть доступен для скачивания и дальнейшего запуска на локальной машине.  


Создаём в домашней директории altlinux name.txt, строку experts:
```
echo  "experts" > ~/name.txt
```

```
nano Dockerfile
```
```
FROM alpine
WORKDIR /hello
COPY name.txt ./
CMD echo "Hello, FIRPO! Greetings from $(cat name.txt)"
```
>FROM - базовый образ  
>WORKDIR - задаёт рабочию директорию внутри контейнера  
>COPY - копирует файл с локального хоста в рабочию директорию контейнера  
>CMD - определяем команду, выполняющаяся после запуска контейнера, после чего контейнер будет остановлен

Сборка образа:
```
docker build -t app .
```
> `-t` имя образа  
> `.` найти Dockerfile в текущей директории  


![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/8aa1a90e-1249-41a2-850c-0c9846ad545a)

Смотрим созданный образ:
```
docker images
```

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/71b41ee4-2a3c-4a9d-9395-09eddd665189)


