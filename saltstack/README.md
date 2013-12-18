OpenStack Heat template: SaltStack cluster (5 nodes)
====================================================

This is an example of HOT(Heat Orchestration Template), which deploy [SaltStack](http://docs.saltstack.com/index.html) cluster, with 1 master node and 5 minion nodes.

You can build it with one command.
```
heat stack-create saltstack \
  -u https://gist.github.com/kjtanaka/7935747/raw/6fcfce4d28b9d5dd7a5b824b678151cc64f522b7/deploy_salt.yml \
  -P "keyname=<key_name>;imagename=<image_name>"
```
