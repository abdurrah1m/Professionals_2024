1. На машине ControlVM.  
&ensp; &ensp; 1. Установите Docker и Docker Compose.  
&ensp; &ensp; 2. Создайте локальный Docker Registry.

# ControlVM

### Заходим под altlinux

Установка:
```
sudo apt-get update && sudo apt-get install -y docker-{ce,compose}
```

Автозагрузка:
```
sudo systemctl enable --now docker.service
```

Права на docker:
```
sudo usermod -aG docker altlinux
```

Выходим `ctrl`+`d` или `exit`

Снова заходим и проверяем

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/68d6ac7c-a456-43bf-a13d-b16372728164)

Docker Registry:
```
docker run -d -p 5000:5000 --restart=always --name DockerRegistry registry:2
```

> порт 5000  
> автозапуск  
> имя DockerRegistry  
> образ registry:2

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/15d0be16-e9d8-4d18-8846-6ae1842db279)
