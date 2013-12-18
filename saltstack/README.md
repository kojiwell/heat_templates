OpenStack Heat template: SaltStack cluster (5 nodes)
====================================================

This is an example of HOT(Heat Orchestration Template), which deploy [SaltStack](http://docs.saltstack.com/index.html) cluster, with 1 master node and 5 minion nodes.

You can build it with one command.
```
heat stack-create saltstack \
  -u https://raw.github.com/kjtanaka/heat_templates/master/saltstack/deploy_salt.yml \
  -P "keyname=<key_name>;imagename=<image_name>"
```
