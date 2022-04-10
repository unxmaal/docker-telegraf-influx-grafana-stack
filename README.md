# Unx's notes

* I wanted to monitor the host running the containers, so I commented out the telegraf section of the docker-compose.yml file, then installed telegraf locally.

```
wget -qO- https://repos.influxdata.com/influxdb.key | sudo tee /etc/apt/trusted.gpg.d/influxdb.asc >/dev/null
source /etc/os-release
echo "deb https://repos.influxdata.com/${ID} ${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
sudo apt-get update && sudo apt-get install telegraf
```
* I then configured /etc/telegraf/telegraf.conf to use the influx container as an 'output'
```
[[outputs.influxdb]]
  urls = ["http://localhost:8086"] # required
  database = "influx" # required
```

* This worked for my local system. 
* Next I wanted to monitor my Mac Pro, so I installed the telegraf agent via 
```
brew install telegraf
```
* I edited /usr/local/etc/telegraf.conf and ensured it had the correct url in outputs.influxdb
* Then restarted the service
```
brew services restart telegraf
```

* I wanted to monitor my Ubiquiti APs next. 
* Install snmp
```
apt install snmp snmpd
```
* Fetch standard mibs
```
apt install snmp-mibs-downloader;sudo download-mibs
```
* Fetch the Ubiquiti MIBs
```
cd /usr/share/snmp/mibs
wget https://dl.ubnt-ut.com/snmp/UBNT-MIB
wget https://dl.ubnt-ut.com/snmp/UBNT-UniFi-MIB

```
* From this repo, https://github.com/WaterByWind/grafana-dashboards/tree/master/UniFi-UAP , I put telegraf-inputs-snmp-uap.conf into /etc/telegraf/telegraf.d/. 

* Then restarted telegraf
```
systemctl restart telegraf
```

* I imported unifi-ap-dashboard.json from the same repo into Grafana as a new dashboard. 
  * Note: this didn't work due to this error, "Failed to upgrade legacy queries e.replace is not a function" 
  * This was a defect in the old version of Grafana in the upstream docker-compose.yml. I bumped the version to 8.4.5 and it works now.

* I restarted the docker-compose stack
```
docker-compose down
docker-compose up
```

* I watched the logs for errors while verifying the dashboards were all working.








-----------------
# Example Docker Compose project for Telegraf, InfluxDB and Grafana

This an example project to show the TIG (Telegraf, InfluxDB and Grafana) stack.

![Example Screenshot](./example.png?raw=true "Example Screenshot")

## Start the stack with docker compose

```bash
$ docker-compose up
```

## Services and Ports

### Grafana
- URL: http://localhost:3000 
- User: admin 
- Password: admin 

### Telegraf
- Port: 8125 UDP (StatsD input)

### InfluxDB
- Port: 8086 (HTTP API)
- User: admin 
- Password: admin 
- Database: influx


Run the influx client:

```bash
$ docker-compose exec influxdb influx -execute 'SHOW DATABASES'
```

Run the influx interactive console:

```bash
$ docker-compose exec influxdb influx

Connected to http://localhost:8086 version 1.8.0
InfluxDB shell version: 1.8.0
>
```

[Import data from a file with -import](https://docs.influxdata.com/influxdb/v1.8/tools/shell/#import-data-from-a-file-with-import)

```bash
$ docker-compose exec -w /imports influxdb influx -import -path=data.txt -precision=s
```

## Run the PHP Example

The PHP example generates random example metrics. The random metrics are beeing sent via UDP to the telegraf agent using the StatsD protocol.

The telegraf agents aggregates the incoming data and perodically persists the data into the InfluxDB database.

Grafana connects to the InfluxDB database and is able to visualize the incoming data.

```bash
$ cd php-example
$ composer install
$ php example.php
Sending Random metrics. Use Ctrl+C to stop.
..........................^C
Runtime:	0.88382697105408 Seconds
Ops:		27 
Ops/s:		30.548965899738 
Killed by Ctrl+C
```

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.

