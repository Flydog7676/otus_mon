# otus_mon
1. Подготовка к установке
- Отключить selinux ``` setenforce 0 ``` и пишем ```sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config```
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
2. Установка Prometheus
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
3. Установка клиента node_exporter
- файл клиента node_exporter ``` cp node_exporter /usr/local/bin/```
- меняем права на файл ```chown node_exporter: /usr/local/bin/node_exporter```
- для автоматического запуска node_exporter сделаем юнит systemd и запустим его ```nano /etc/systemd/system/node_exporter.service```
- заполняем юнит
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
- перечитываем конфигурацию systemd юнитов ```systemctl daemon-reload```
- запускаем юнит и разрешаем его автозапуск 
```
systemctl start node_exporter
systemctl enable node_exporter
```
- прописываем наличие node_exportera в prometheus ``` nano /etc/prometheus/prometheus.yml```
- добавляем в конце 
```
- job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```
- перезапускаем prometheus ```systemctl restart prometheus```
4. Установливаем grafana
- устанавливаем ранее скачанный пакет ``` yum install grafana-8.1.1-1.x86_64.rpm```
- запускаем grafana и разрешаем его автозапуск 
```
systemctl start grafana-server
systemctl enable grafana-server
```
5. Настраиваем grafana на сбор статистики и отображение информации
- находим наш адрес ip ``` ip a```
- заходим на страницу Grafana из броузера ``` http://192.168.11.101:3000```
- добавляем новый Data Sources ``` http://localhost:9090```
- импортируем дашбоард к примеру 11074
