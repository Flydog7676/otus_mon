# otus_mon
1. Подготовка к установке
- Отключить selinux ``` setenforce 0 ``` и в файле ```/etc/selinux/config``` пишем ```SELINUX=permissive```
- создаем технических пользователей
```
useradd --no-create-home --shell /usr/sbin/nologin prometheus
useradd --no-create-home --shell /bin/false node_exporter
```
- создание необходимых директорий ```mkdir /etc/prometheus /var/lib/prometheus```
- смена прав на созданные директории на пользователя ```chown -Rv prometheus: /var/lib/prometheus/ /etc/prometheus/```
- скачиваем нужные дистрибутивы программ prometheus, node_exporter, grafana
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
wget https://dl.grafana.com/oss/release/grafana-8.1.1-1.x86_64.rpm
```
- распаковываем скачанные дистрибутивы
```
tar xvzf node_exporter-1.2.2.linux-amd64.tar.gz
tar xvzf prometheus-2.29.1.linux-amd64.tar.gz 
```
2. Установка нужных программ
- копируем из папки prometheus
```
cp -r console_libraries consoles prometheus.yml /etc/prometheus
cp prometheus promtool /usr/local/bin/
```
- меняем права на папки  
 ``` 
 chown -R prometheus: /etc/prometheus /var/lib/prometheus
 chown prometheus: /usr/local/bin/{prometheus,promtool}
 ```
- для автоматического запуска prometheus сделаем юнит systemd и запустим его ```nano /etc/systemd/system/prometheus.service```
- заполняем юнит
```
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```
- перечитываем конфишурацию systemd юнитов ```systemctl daemon-reload```
- запускаем юнит и разрешаем его автозапуск 
```
systemctl start prometheus
systemctl enable prometheus
```
