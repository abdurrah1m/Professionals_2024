```
nano docker-compose.yml
```

```yaml
version: '3.7'
services:

  zabbix-db:
    image: postgres:15
    environment:
      POSTGRES_DB: "zabbix"
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix"
    restart: always

  zabbix-server:
    image: zabbix/zabbix-server-pgsql:ubuntu-latest
    environment:
      DB_SERVER_HOST: "zabbix-db"
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix"
      POSTGRES_DB: "zabbix"
    depends_on:
      - "zabbix-db"
    links:
      - "zabbix-db"
    restart: always

  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql:ubuntu-latest
    environment:
      DB_SERVER_HOST: "zabbix-db"
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix"
      POSTGRES_DB: "zabbix"
      ZBX_SERVER_HOST: "zabbix-server"
      PHP_TZ: "Asia/Yekaterinburg"
    depends_on:
      - "zabbix-db"
      - "zabbix-server"
    links:
      - "zabbix-db"
      - "zabbix-server"
    ports:
      - "80:8080"
    restart: always
```

`zabbix-db` - база данных postgres15. table - `zabbix`, user - `zabbix`, password - `zabbix`  
`zabbix-server` - сам zabbix.  
`zabbix-web` - веб-морда.  

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/66fc6c08-d7fc-4cc1-be05-f492cd492edd)
