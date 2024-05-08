### Fuentes,tutoriales y archivos de configuracion

Proyecto blackbox_exporter
https://github.com/prometheus/blackbox_exporter?tab=readme-ov-file

Fuente Black exporter
https://github.com/prometheus/blackbox_exporter/releases

Ejemplo de pasos de configuracion step by step

`<link english>` : <https://www.youtube.com/watch?v=HbaiglWbhR0>

`<link spanish>` : <https://www.youtube.com/watch?v=yW0t029OIEw>


### Instalación y configuración

1-Archivos y permisos
2-Puertos de SO
3-Configuración de exporters
4-Activacación de exporters, prometheus, grafana
5-Configuración de dashboards

####1-Archivos y permisos

En el directorio donde se guardan los archivos y programas de monitoreo, se modifica los permisos de usuario, descargar los programas. Descomprimir los archivos tar.

Permisos del filesystem

```sh
chmod 755 .
```

Descarga Grafana
```sh
wget https://dl.grafana.com/oss/release/grafana-10.4.2.linux-amd64.tar.gz
tar -zxvf grafana-10.4.2.linux-amd64.tar.gz
```

Descarga Prometheus
```sh
wget https://github.com/prometheus/prometheus/releases/download/v2.51.2/prometheus-2.51.2.linux-amd64.tar.gz
tar -zxvf prometheus-2.51.2.linux-amd64.tar.gz
```

Descarga Black_exporter
```sh
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar -zxvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
```

Descarga Node_exporter
```sh
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz
tar -zxvf node_exporter-1.8.0.linux-amd64.tar.gz
```

Descarga Pushgateway
```sh
wget https://github.com/prometheus/pushgateway/releases/download/v1.8.0/pushgateway-1.8.0.linux-amd64.tar.gz
tar -zxvf pushgateway-1.8.0.linux-amd64.tar.gz
```


2- Puertos de SO (Linux)

Cada programa hace uso de un puerto especifico donde se comunican hacia prometheus o entre ellos. En la siguiente imagen se explica la comunicación. 


```sh
sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp
sudo firewall-cmd --permanent --zone=public --add-port=3100/tcp
sudo firewall-cmd --permanent --zone=public --add-port=9100/tcp
sudo firewall-cmd --permanent --zone=public --add-port=9154/tcp
sudo firewall-cmd --permanent --zone=public --add-port=9154/tcp
sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp

firewall-cmd --reload
```

Anexo

Proyecto blackbox_exporter
https://github.com/prometheus/blackbox_exporter?tab=readme-ov-file

Fuente Black exporter
https://github.com/prometheus/blackbox_exporter/releases

Ejemplo de pasos de configuracion step by step

`<link english>` : <https://www.youtube.com/watch?v=HbaiglWbhR0>
`<link spanish>` : <https://www.youtube.com/watch?v=yW0t029OIEw>
