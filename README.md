# ansible-en-role

Ansible role to deploy and configure zkSync Era External Node, including DB isntance setup on the same machine, Traefik as reverse proxy, and Prometheus monitoring (PostgreSQL exporter, Node exporter, cAdvisor, Traefik, External Node native metrics, and VictoriaMetrics vmagent to scrape all of them).

Make sure to configure Prometheus remote write endpoint to send metrics to centralized metrics storage.

Role has been tested and used internally on bare metal Hetzner instances.

## Requirements

This role has been tested on:

* Ubuntu 22.04, Jammy Jellyfish; Ansible 2.13.9

## Usage

Minimal required variables that has to be set:

```yaml
database_name: ""
database_username: ""
database_password: ""
eth_l1_url: ""
main_node_url: ""
l1_chain_id: ""
l2_chain_id: ""
```

If you want to use monitoring (which we highly recommend), you have to change these variables:

```yaml
# Monitoring options section
enable_monitoring: true
node_name: "some-unique-node-identifier"
prometheus_remote_write: true
prometheus_remote_write_url: "https://metrics.example.org"
prometheus_remote_write_auth: true
prometheus_remote_write_auth_username: "admin"
prometheus_remote_write_auth_password: "password"
prometheus_remote_write_common_label: "matterlabs"
```

This role also has option to secure your server and allow traffic only from specified IP address in case if you want
to use some load balancer in front of your node, while not having fancy cloud security groups at your disposal:

```yaml
# Security options
use_predefined_iptables: true
disable_ssh_password_auth: true
iptables_packages:
  - iptables
  - iptables-persistent
# Variable to be used to accept external traffic only from single specified IP
loadbalancer_ip: "1.2.3.4"
```

In most of cases, you'd want to change PostgreSQL parameters (we recommend to use <https://pgtune.leopard.in.ua/> with "Online transaction processing system" preset as sane defaults), so you can do it using `postgres_arguments` variable, eg:

```yaml
postgres_arguments:
  - log_error_verbosity=terse
  - -c
  - max_connections=256
  - -c
  - shared_buffers=47616MB
  - -c
```

We recommend using pgtune [online]<https://pgtune.leopard.in.ua/> or [self-hosted](https://github.com/le0pard/pgtune) version with with "Online transaction processing system" preset as a good starting point for generating optimal config for your hardware.

## Step-by-step guide

1. Install ansible collection on your machine from where you will run ansible:
`ansible-galaxy collection install community.general`

2. Prepare latest database backup on your host. you can download it from our [public GCS bucket](https://storage.googleapis.com/zksync-era-mainnet-external-node-backups/external_node_latest.pgdump).
you should place it to `{{ storage_directory }}/pg_backups` directory. By default, `{{ storage_directory }}` is `/usr/src/en`

3. **OPTIONAL**: If you already have external-node, you can copy tree directory to new host. Copy external-node database tree to `{{ storage_directory }}/db`.

**Keep in mind, tree should be older than PostgreSQL database backup.**

4. Run ansible-playbook using this role. We recommend to encrypt next variables with ansible-vault or some another way:

```
database_username
database_password
eth_l1_url
vm_auth_username
vm_auth_password
```

5. Connect to your host, and see status of `postgres` container. It can take a lot of time before PostgreSQL database backup will be restored (hours to days, depending on your disk throughput and IOPS), after which PostgreSQL server will be ready for use. Once `postgres` becomes "healthy", `external_node` runs automatically.

## Example Playbook

```yaml
---
- hosts: all
  become: true
  vars:
    loadbalancer_ip: "1.2.3.4"
    use_predefined_iptables: true
    enable_monitoring: false
    database_name: "mainnet2"
    main_node_url: "https://zksync2-mainnet.zksync.io"
    l2_chain_id: "324"
    l1_chain_id: "1"
    enable_tls: false
    partner_id: matterlabs
  vars_files:
    - secrets/mainnet_secrets.yml
  roles:
    - external_node
```

## License

Ansible role for zkSync Era External Node is distributed under the terms of either

* Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
* MIT license ([LICENSE-MIT](LICENSE-MIT) or <https://opensource.org/blog/license/mit/>)

at your option.
