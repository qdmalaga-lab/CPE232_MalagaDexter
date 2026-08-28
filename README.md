# CPE_MIDEXAM_MALAGA

Ansible Infrastructure-as-Code project for the CPE 232 Midterm Skills Exam:
Install, Configure, and Manage Log Monitoring Tools.

## What this deploys

| Stack | Hosts (from inventory group) | What gets installed |
|---|---|---|
| Elastic Stack | `elasticsearch`, `kibana`, `logstash` (3 separate hosts) | Elasticsearch, Kibana, Logstash |
| Nagios | `nagios` (1 host) | Nagios Core + web UI |
| Monitoring | `influxdb`, `prometheus`, `grafana` (3 separate hosts) | InfluxDB 2.x, Prometheus, Grafana |
| LAMP | `web`, `db` (2 separate hosts) | Apache + PHP, MariaDB |

## Project structure

```
CPE_MIDEXAM_MALAGA/
├── ansible.cfg
├── config.yaml              # central variables consumed by every playbook
├── requirements.yml         # required Galaxy collections
├── inventory/
│   └── hosts.ini            # target hosts, grouped by service
├── playbooks/
│   ├── site.yml             # master playbook - runs everything
│   ├── elastic_stack.yml
│   ├── nagios.yml
│   ├── monitoring_stack.yml
│   └── lamp_stack.yml
└── roles/
    ├── elasticsearch/
    ├── kibana/
    ├── logstash/
    ├── nagios/
    ├── influxdb/
    ├── prometheus/
    ├── grafana/
    ├── apache_php/
    └── mariadb/
```

## Prerequisites

1. Ansible control node with `ansible-core` installed.
2. Target VMs reachable over SSH, with a sudo-capable user (matches
   `ansible_user` in `inventory/hosts.ini`).
3. Passwordless SSH key auth set up from the control node to every target
   (see Activity 8 for the key-based auth workflow).
4. Required collections installed:
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```

## Before running

Edit `inventory/hosts.ini` and replace every `ansible_host` IP with your
actual target VM addresses. Edit `config.yaml` to change any
versions/ports/credentials as needed.

## Usage

Run everything at once:
```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yml -e "@config.yaml"
```

Or run one stack at a time (useful for testing/screenshots one piece at a time):
```bash
ansible-playbook -i inventory/hosts.ini playbooks/elastic_stack.yml -e "@config.yaml"
ansible-playbook -i inventory/hosts.ini playbooks/nagios.yml -e "@config.yaml"
ansible-playbook -i inventory/hosts.ini playbooks/monitoring_stack.yml -e "@config.yaml"
ansible-playbook -i inventory/hosts.ini playbooks/lamp_stack.yml -e "@config.yaml"
```

Always syntax-check before a real run:
```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yml --syntax-check
```

Dry-run (no changes made) to preview what would happen:
```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yml -e "@config.yaml" --check
```

## Verifying each service after install

| Service | Check |
|---|---|
| Elasticsearch | `curl http://<es-host>:9200/_cluster/health` |
| Kibana | Browse to `http://<kibana-host>:5601` |
| Logstash | `sudo systemctl status logstash` |
| Nagios | Browse to `http://<nagios-host>/nagios` (login: nagiosadmin) |
| InfluxDB | Browse to `http://<influxdb-host>:8086` |
| Prometheus | Browse to `http://<prometheus-host>:9090/targets` |
| Grafana | Browse to `http://<grafana-host>:3000` (login: admin) |
| Apache/PHP | Browse to `http://<web-host>/info.php` |
| MariaDB | `mysql -u appuser -p -h <db-host> appdb` |
