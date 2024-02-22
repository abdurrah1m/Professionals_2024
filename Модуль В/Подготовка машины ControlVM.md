# Подготовка машины ControlVM  

&ensp; 1. Вся проверка выполнения задания будет проводиться с машины ControlVM.  
&ensp; 2. НЕ удаляйте ControlVM по завершении задания.  
&ensp; 3. Создайте инстанс с именем ControlVM и подключите его к сети интернет.  
&ensp; &ensp; 1. Тип виртуальной машины: 2 vCPU, Доля vCPU 50%, 2 RAM.  
&ensp; &ensp; 2. Размер диска: 10 ГБ.  
&ensp; &ensp; 3. Тип диска: SSD.  
&ensp; &ensp; 4. Отключите мониторинг и резервное копирование.  
&ensp; &ensp; 5. Операционная система: ALT Linux 10.  
&ensp; &ensp; 6. Разрешите внешние подключения по протоколу SSH.  
&ensp; &ensp; 7. Сохраните ключевую пару для доступа на рабочем столе вашего локального ПК с расширением .pem  
&ensp; 4. Настройте внешнее подключение к ControlVM.  
&ensp; &ensp; 1. Установите на локальный ПК клиент SSH PuTTY.  
&ensp; &ensp; 2. Создайте в PuTTY профиль с именем YACloud.  
&ensp; &ensp; 3. Убедитесь в возможности установления соединения с ControlVM с локального ПК с помощью клиента PuTTY без ввода дополнительных параметров.  
&ensp; &ensp; 4. Используйте для подключения имя пользователя altlunxu и загруженную ключевую пару.  

# Буду использовать Mobaxterm, который базирован на putty

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/d5e36f86-9a5e-452b-87e4-88b44ed16b4a)

Tools - MobaKeyGen

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/363a48d1-5411-4440-aab4-a26d336ef793)

Generate - копируем публичный ключ - Save private key - меняем расширение ключа на `pem`

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/e4390e4b-c7d2-4ed7-b887-956e73d2e6f6)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/6aebff10-f997-4026-bb77-27a56759f02e)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/99cc5ea4-1fd4-45b9-93d5-94dfa01ea772)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/4ee74e84-096a-4a58-91ee-5070bea69f67)
