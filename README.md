# otus_mon
1. Подготовка к установке
- Отключить selinux ``` setenforce 0 ``` и в файле ```/etc/selinux/config``` пишем ```SELINUX=permissive```
- создаем технических пользователей
```
useradd --no-create-home --shell /usr/sbin/nologin prometheus
useradd --no-create-home --shell /bin/false node_exporter
```
- создание необходимых директорий
```
mkdir /etc/prometheus
mkdir /var/lib/prometheus
```
- смена прав на созданные директории на пользователя ```chown -Rv prometheus: /var/lib/prometheus/ /etc/prometheus/```
- скачиваем нужные дистрибутивы программ prometheus, node_exporter, grafana
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.darwin-amd64.tar.gz
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.darwin-amd64.tar.gz
wget https://dl.grafana.com/oss/release/grafana-8.1.1-1.x86_64.rpm
```
- распаковываем скачанные дистрибутивы
```
tar xvzf node_exporter-1.2.2.darwin-amd64.tar.gz
tar xvzf prometheus-2.29.1.darwin-amd64.tar.gz 
```
2. Установка нужных программ
- копируем из папки prometheus
```
cp -r console_libraries consoles prometheus.yml /etc/prometheus
cp prometheus promtool /usr/local/bin/
```
- меняем права на папки   ``` chown -Rv prometheus: /usr/local/bin/prom{etheus,tool} /etc/prometheus/ ```
- 
