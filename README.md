# ISA Monitoring

## But

Le but est de remonter des métriques du scoreboard d'Apache dans Prometheus et Grafana.

## Schéma

<img src="doc/isa-monitoring.png" alt="architecture" width="500"/>

### Apache

Apache doit avoir le module status.

```
[root@isaqal09 ~]# apachectl -t -D DUMP_MODULES | grep status 
 status_module (shared)
Syntax OK
```

La configuration d'Apache est modifiée afin de pouvoir avoir:

```
[root@isaqal09 ~]# curl http://localhost/server-status?auto 
Total Accesses: 6111
Total kBytes: 28367
CPULoad: .0694061
Uptime: 20272
ReqPerSec: .30145
BytesPerSec: 1432.9
BytesPerReq: 4753.36
BusyWorkers: 1
IdleWorkers: 10
Scoreboard: _____W_____...................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
```

### Apache exporter

[Apache exporter](https://github.com/Lusitaniae/apache_exporter) a pour but de scraper les données d'Apache à l'adresse `localhost/server-status?auto` et de préparer les métriques au format souhaité par Prometheus.

'Apache exporter' tourne dans un conteneur docker.

On peut se convaincre que apache_exporter fonctionne convenablement. 
Pour cela, on se connecte au conteneur `apache_exporter` 

`docker exec -it apache_exporter sh`

Puis:

```
# wget localhost:9117/metrics
Connecting to localhost:9117 (127.0.0.1:9117)
saving to 'metrics'
metrics              100% |*****************************************************************************************************************************************|  7831  0:00:00 ETA
'metrics' saved
```

et enfin:

```
/ # cat metrics 
....
# TYPE apache_scoreboard gauge
apache_scoreboard{state="closing"} 0
apache_scoreboard{state="dns"} 0
apache_scoreboard{state="graceful_stop"} 0
apache_scoreboard{state="idle"} 10
apache_scoreboard{state="idle_cleanup"} 0
apache_scoreboard{state="keepalive"} 0
apache_scoreboard{state="logging"} 0
apache_scoreboard{state="open_slot"} 1013
apache_scoreboard{state="read"} 0
apache_scoreboard{state="reply"} 1
apache_scoreboard{state="startup"} 0
...
```

### Prometheus

On peut accéder à l'interface de Prometheus à l'adresse suivante `https://isaqal09.epfl.ch:8443/`

#### Targets

`https://isaqal09.epfl.ch:8443/targets`

<img src="doc/prometheus-targets.png" alt="Prometheus Targets" width="400"/>

#### Queries

`https://isaqal09.epfl.ch:8443/graph?g0.range_input=1h&g0.expr=apache_scoreboard&g0.tab=1`

<img src="doc/prometheus-queries.png" alt="Prometheus Queries" width="400"/>

Prometheus tourne dans un conteur docker.

### Traefik

On utilise traefik principalement pour mettre en place une "basic auth".

Le nom de l'utilisateur et le mot de passe sont stockés dans keybase.

TODO: Ajouter les bonnes personnes (qui?) dans la team epfl_isa via son email

### Grafana

On utilise le "Grafana as a service" de c2c à l'adresse https://grafana-exopge.epfl.ch.
Notre organisation a pour nom "EPFL IS-Academia".

Dans l'application groups.epfl.ch, le group [grafana-exopge-isacademia-org-editors](https://groups.epfl.ch/cgi-bin/groups/viewgroup?groupid=S27146) permet de définir les utilisateurs qui peuvent gérer des dashboards.

TODO: Ajouter les bonnes personnes (qui?) dans ce groupe

## Ansible

Nous utilisons [Ansible](https://www.ansible.com/) pour mettre en place les conteneurs Docker et l'ensemble de la configuration nécessaire (configuration apache, gestion IPTables, certificat SSL, etc).
Il s'agit donc d'une "configuration as code"

### Prérequis 

- Installer ansible
- Récupérer les sources de https://github.com/epfl-si/isa-monitoring

### Déployement

La commande ci-dessous vous permet de déployer l'ensemble de la configuration sur les serveurs 'qualité' c'est à dire les hosts `isaqal09` et `isaqal10`

```
./ansible/isasible
```

La commande ci-dessous vous permet de déployer l'ensemble de la configuration sur `isaqal09` uniquement
```
./ansible/isasible -l isaqal09
```

Pour déployer sur les serveurs de production, il faudra modifier le fichier:
`ansible/inventory/hosts-prod.yml` => Voir #TODO

```
./ansible/isasible --prod
```
