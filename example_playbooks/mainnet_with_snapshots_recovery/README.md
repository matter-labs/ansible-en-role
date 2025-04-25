# Mainnet Snapshots Recovery playbook

This directory is simple example how to set up EN using this role. It comes with snapshots recovery enabled by default.\
**Note that for simplicity it's using postgres database
with a very unsecure password and the EN is just started on the same machine**

To run this playbook, first install dependencies

```shell
 ansible-galaxy install -r requirements.yml
 ```

and then you can run the playbook using

```shell
 ansible-playbook playbook.yml -i hosts.ini -K
 ```

To see logs you can use

```shell
 docker logs en-external_node-1
 ```
