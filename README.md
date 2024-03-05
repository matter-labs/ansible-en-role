# ansible-en-role
Ansible role for setup external node.

## Requirements
This role has been tested on:
* Ubuntu 22.04, Jammy Jellyfish; Ansible 2.13.9

## Usage
This role contains variables which has to be set:
```yaml
database_name: ""
database_username: ""
database_password: ""
eth_l1_url: ""
main_node_url: ""
l1_chain_id: ""
l2_chain_id: ""
```

If you want to use monitoring, you can use next variables:
```yaml
# Monitoring options section
enable_monitoring: false
node_name: ""
prometheus_remote_write: false
prometheus_remote_write_url: ""
prometheus_remote_write_auth: false
prometheus_remote_write_auth_username: ""
prometheus_remote_write_auth_password: ""
prometheus_remote_write_label: ""
```

This role also has option to secure your server and allow traffic only from specified ip in case if you want
to use some load balancer in front of your node:

```yaml
# Security options
use_predefined_iptables: false
disable_ssh_password_auth: false
iptables_packages:
  - iptables
  - iptables-persistent
# Variable can be used in case with accept external traffic only from one ip
loadbalancer_ip: ""
```

In some cases, you may need to change postgres parameters, so you can do it using `postgres_arguments` variable:
```yaml

postgres_arguments:
  - log_error_verbosity=terse
  - -c
  - max_connections=256
  - -c
  - shared_buffers=47616MB
  - -c
  - effective_cache_size=142848MB
  - -c
  - maintenance_work_mem=2GB
  - -c
  - checkpoint_completion_target=0.9
  - -c
  - wal_buffers=16MB
  - -c
  - default_statistics_target=500
  - -c
  - random_page_cost=1.1
  - -c
  - effective_io_concurrency=200
  - -c
  - work_mem=2573kB
  - -c
  - huge_pages=try
  - -c
  - min_wal_size=4GB
  - -c
  - max_wal_size=16GB
  - -c
  - max_worker_processes=74
  - -c
  - max_parallel_workers_per_gather=37
  - -c
  - max_parallel_workers=74
  - -c
  - max_parallel_maintenance_workers=4
  - -c
  - checkpoint_timeout=1800
```
We recommend to use [pgtune](https://github.com/le0pard/pgtune) to choose optimal config for your hardware. 

## Step-by-step guide

1. Install ansible collection on your machine from where you will run ansible:
`ansible-galaxy collection install community.general`
2. Prepare latest database backup on your host. you can download it from our [public GCS bucket](https://storage.googleapis.com/zksync-era-mainnet-external-node-backups/external_node_latest.pgdump).
you should place it to `{{ storage_directory }}/pg_backups` directory. By default, `{{ storage_directory }}` is `/usr/src/en` 
3. **OPTIONAL**: If you already have external-node, you can copy tree directory to new host. Copy external-node database tree to `{{ storage_directory }}/db`. 
**Keep in mind, tree should be older than postgres database backup.**
4. Run ansible-playbook using this role. We recommend to encrypt next variables with ansible-vault or some another way:
```
database_username
database_password
eth_l1_url
vm_auth_username
vm_auth_password
```
5. Connect to your host, and see status of postgres container. It can take a lot of time before postgres database backup will be restored 
and postgres server will be ready for use. After postgres goes healty status, external-node runs automatically.

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

Ansible role for external node is distributed under the terms of either

- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or <https://opensource.org/blog/license/mit/>)

at your option.
