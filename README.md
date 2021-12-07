# prometheus-mongo-exporter-grafana
Mongo veritabanındaki metriklerin Mongo-Exporter ile Prometheus'un çektiği ve Grafanada görüntülenmesi üzerine konteyner

Tüm konteynerleri >
Çalıştırmak için:

```shell
docker-compose -f "_PROMETHEUS\docker-compose.yml" up -d --build 
```

Durdurmak için:

```shell
docker-compose -f "_PROMETHEUS\docker-compose.yml" down
```

Tek tek >
Mongo veritabanını ayaklandırmak için
```shell
docker run -p 27017:27017 --name cmongodb mongo
```

Mongo metriklerini ihraç edebilecek NodeExporter'ı ayaklandırmak için
```shell
docker run --rm --name prom-me \
       -p 9123:9123 \
       noenv/mongo-exporter \
         --web.listen-address=:9123 \
         --mongodb.uri=mongodb://172.17.0.3:27017/?tls=false&tlsInsecure=true
```
Mongo metriklerini veritabanından çekerek Prometheus'un çekebileceği hale getiren Node Exporter 9123 portunda çalışıyor ve internet gezgininden http://localhost:9123/ ve http://localhost:9123/metrics adresilerinden erişilebilinir.

Prometheus'u ayaklandırmak için 
```shell
docker service create --replicas 1 --name my-prometheus
   --mount type=bind,source=C:/tmp/prometheus.yml,destination=/etc/prometheus/prometheus.yml
   --publish published=9090,target=9090,protocol=tcp
   prom/prometheus
```

Sadece Grafayı ayaklandırmak için:
```shell
 docker run -d \
     -p 3000:3000 \
     --name=grafana \
     -e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource" \
     grafana/grafana
```

Grafana'yı başlatmak için http://localhost:3000 adresine (kullanıcı adı ve şifresi "admin" ile) girilir ve Data Source kısmına Prometheus eklenerek Prometheus'un konteynerler arasında erişilebileceği URL adresi (http://cprom:9090) yazılarak "Save & Test" düğmesine basılarak eklenir.

### Metrik adını değiştirmek
```
    metric_relabel_configs:
    - source_labels: [__name__]
      regex: mongodb_ss_catalogStats_(.*)
      target_label: __name__
      replacement: "cem_${1}"
      action: replace
    - source_labels: [__name__]
      regex: AttSessionImOrig
      target_label: __name__
      replacement: "attSession_im_orig"
      action: replace
```

mongodb_ss_catalogStats_capped Metriğinin "job" özelliğinin içeriğini değiştirmek
Önce sadece metrik adı değişik gelirken           : cem_capped{instance="cexp:9123", job="prometheus", rs_state="0"}
15sn Sonra metriği çekince job içeriği de değişir : cem_capped{instance="cexp:9123", job="cem_job_", rs_state="0"}
```
    - source_labels: [cem_capped]
      regex: (.*)
      target_label: jem
      replacement: "cem_job_${1}"
      action: replace
```
