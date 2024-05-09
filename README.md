## Introducción

Este proyecto explica la instalación, configuracion y explotacion de las herramientas como Grafana, Prometheus y influxdb, ademas de los exporters que son utilizados para enviar parametros custom(pushgateway) o existentes como node_exporter,black_exporter, etc. Este repositorio esta dividido en pasos de instalacion y configuracion, PDF con paso a paso para replicar el lab.

### Datos tecnicos

SO: Oracle Linux 5.4.17-2136.329.3.1.el8uek.x86_64 (OracleLinux-R8-U5-x86_64) https://yum.oracle.com/oracle-linux-isos.html

Paquetes de SO: telnet,curl
VirtualBox: V7.0
Memoria RAM 8gb
Tarjeta de red virtual: Adapter 2 host-only Adapter, Adapter#2

### Arquitectura de la solucion 

La solucion se compone de la siguiente manera, realizar una comunicacion mediante puertos de red, la exportacion de datos de cada exporter o custom script es enviada al tsdb (time serie-database) en este caso prometheus, grafana consulta o exporta los datos mediante una conexion de datasource 

![](https://pandao.github.io/editor.md/images/ArquitecturaGeneral.png)

1-Cluster de servidores

Ya sea una solucion stand-alone o en cluster, los servidores tendria dentro los exporter ejecutandose como programas, en este caso seria pushgateway, procesos jobs y node_exporter

2-Pushgateway actua como un recolector de datos mediante un endpoint, los batch jobs ejecutan el script donde se recolecta datos y son enviados hacia pushgateway, este ultimo envia a prometheus los datos generados. Node_exporter es un proceso que contiene codigo y librerias que recolecta metricas del SO, estos datos son enviados al prometheus.

3-Prometheus cumple la funcion de almacenar y mantener las metricas recolectadas de los exporters. Mediante un datasource en grafana tiene acceso a las metricas exportas. El tsdb es una base de datos de serie de tiempo optimizada para programas de tipo IoT que envia metricas a una alta velocidad, es una base de datos no relacional [Más información](https://prometheus.io/docs/prometheus/latest/storage/)

4-Grafana es un vizualizador de datos mediante dashboard custom o personalizados, mediante el datasource se conecta al prometheus exportando metricas con el uso del lenguaje PromQL.

![](https://pandao.github.io/editor.md/images/ArquitecturaPrometheus.png)

### Instalación y configuración

1-Archivos y permisos
2-Puertos de SO
3-Configuración de prometheus y grafana
4-Activación de exporters, prometheus, grafana
5-Configuración de dashboards

####1-Archivos y permisos

En el directorio donde se guardan los archivos y programas de monitoreo, se modifica los permisos de usuario, descargar los programas. Descomprimir los archivos tar.
En este caso se uso el directorio /home/oracle/grafana pero se puede crear un filesystem en la / del SO durante la instalación de linux

Permisos del filesystem

```sh
unmask 022
cd /home/oracle
mkdir grafana
chmod 755 grafana
```

Descarga Grafana
```sh
wget https://dl.grafana.com/oss/release/grafana-10.4.2.linux-amd64.tar.gz
tar -zxvf grafana-10.4.2.linux-amd64.tar.gz
mv grafana-10.4.2.linux-amd64.tar.gz grafana_10_4
```

Descarga Prometheus
```sh
wget https://github.com/prometheus/prometheus/releases/download/v2.51.2/prometheus-2.51.2.linux-amd64.tar.gz
tar -zxvf prometheus-2.51.2.linux-amd64.tar.gz
mv prometheus-2.51.0.linux-amd64 prometheus
```

Descarga Black_exporter
```sh
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar -zxvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
mv blackbox_exporter-0.24.0 black_Exporter
```

Descarga Node_exporter
```sh
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz
tar -zxvf node_exporter-1.8.0.linux-amd64.tar.gz
mv node_exporter-1.8.0.linux-amd64 node_exporter
```

Descarga Pushgateway
```sh
wget https://github.com/prometheus/pushgateway/releases/download/v1.8.0/pushgateway-1.8.0.linux-amd64.tar.gz
tar -zxvf pushgateway-1.8.0.linux-amd64.tar.gz
mv pushgateway-1.8.0.linux-amd64 pushgateway_exporter
```

2- Puertos de SO (Linux)

Cada programa hace uso de un puerto especifico donde se comunican hacia prometheus o entre ellos. En la siguiente imagen se explica la comunicación. 


```sh
sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp #port-grafana
sudo firewall-cmd --permanent --zone=public --add-port=9090/tcp #port prometheus
sudo firewall-cmd --permanent --zone=public --add-port=9100/tcp #port node_exporter
sudo firewall-cmd --permanent --zone=public --add-port=9154/tcp #port black_exporter
sudo firewall-cmd --permanent --zone=public --add-port=9091/tcp #port pushgateway
sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp 

firewall-cmd --reload
```

3-Configuracion de prometheus

Buscar el archivo prometheus.yml en directorio /home/oracle/grafana/prometheus, agregar los job_name que corresponde a cada exporter, en el caso de prometheus se viene cambiar la ip y puerto donde corre para acceder mediante el browser. El siguiente ejemplo viene para servidores en cluster, el black_exporter es diferente, se debe especificar los endpoint o url que requiere monitoreo.

```yml
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label job=<job_name> to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["192.168.100.2:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["192.168.100.2:9100","192.168.100.4:9100"]

  - job_name: "pushgateway"
    static_configs:
      - targets: ["192.168.100.2:9091","192.168.100.4:9091"]

  - job_name: 'blackbox_exporter'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://google.com
        - http://prometheus.io
        - https://prometheus.io
    relabel_configs:
      - source_labels: [_address_]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: _address_
        replacement: 192.168.100.2:9115  # The blackbox exporter's real hostname:port.
```

Modificar archivo 

Colocar la ip y puerto donde levanta el grafana. Nota: Si quieres cambiar el puerto agregarlo al firewall del SO. Tambien puedes subirlo en localhost

```sh
cd /home/oracle/grafana/grafana10_4/conf
nano sample.ini
```

4-Activación de exporters, prometheus, grafana

Prometheus,node_exporter,black_exporter,pushgateway,grafana.



5-Configuración de dashboards

IMAGEN
![](https://pandao.github.io/editor.md/images/ArchivoSampleGrafana.png)

Anexo

Proyecto blackbox_exporter
https://github.com/prometheus/blackbox_exporter?tab=readme-ov-file

Fuente Black exporter
https://github.com/prometheus/blackbox_exporter/releases

Ejemplo de pasos de configuracion step by step

`<link english>` : <https://www.youtube.com/watch?v=HbaiglWbhR0>
`<link spanish>` : <https://www.youtube.com/watch?v=yW0t029OIEw>
