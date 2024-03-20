# Docker - NodeExporter, Prometheus и Grafana

&ensp; &ensp; 7. Настройте мониторинг с помощью NodeExporter, Prometheus и Grafana.  
&ensp; &ensp; &ensp; 1. Создайте в домашней директории пользователя ubuntu файл monitoring.yml для Docker Compose  
&ensp; &ensp; &ensp; &ensp; 1. Используйте контейнеры NodeExporter, Prometheus и Grafana для сбора, обработки и отображения метрик.  
&ensp; &ensp; &ensp; &ensp; 2. Настройте Dashboard в Grafana, в котором будет отображаться загрузка CPU, объём свободной оперативной памяти и места на диске.  
&ensp; &ensp; &ensp; &ensp; 3. Интерфейс Grafana должен быть доступен по внешнему адресу на порту 3000.  

В домашней директории юзера altlinux `/home/altlinux`:
```
nano monitoring.yml
```
```
version: "3.9"
services:
  grafana:
    container_name: Grafana
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - grafana-configs:/etc/grafana

  prometheus:
    container_name: Prometheus
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - prom-data:/prometheus
      - prom-configs:/etc/prometheus

  node-exporter:
    container_name: NodeExporter
    image: prom/node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude'
      - '^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)'
volumes:
  grafana-data:
  grafana-configs:
  prom-data:
  prom-configs:
```
