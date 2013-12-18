README
======

1. Create Hadoop cluster

```
heat stack-create hadoop -u https://raw.github.com/kjtanaka/heat_templates/master/hadoop/deploy_hadoop.yml \
    -P "key_name=<key name>;image_name=<image name>"
```

2. Register hdfs's authorized_keys

```
scp -p hdfs@<node01's ip>:.ssh/authorized_keys hdfs_auth_keys
scp -p hdfs_auth_keys hdfs@<node02's ip>:.ssh/authorized_keys
scp -p hdfs_auth_keys hdfs@<node03's ip>:.ssh/authorized_keys
scp -p hdfs_auth_keys hdfs@<node04's ip>:.ssh/authorized_keys
scp -p hdfs_auth_keys hdfs@<node05's ip>:.ssh/authorized_keys
```

3. Copy /etc/hosts from node01 to the others

```
ssh hdfs@<node01's ip>
hdfs@node01$ cat /etc/hosts | ssh node02 'sudo sh -c "cat > /etc/hosts"'
hdfs@node01$ cat /etc/hosts | ssh node03 'sudo sh -c "cat > /etc/hosts"'
hdfs@node01$ cat /etc/hosts | ssh node04 'sudo sh -c "cat > /etc/hosts"'
hdfs@node01$ cat /etc/hosts | ssh node05 'sudo sh -c "cat > /etc/hosts"'
```

4. Format HDFS

```
hdfs@node01$ hadoop namenode -format
```

5. Start Hadoop

```
hdfs@node01$ ./bin/start-all.sh
```

6. Stop Hadoop

```
hdfs@node01$ ./bin/stop-all.sh
```
