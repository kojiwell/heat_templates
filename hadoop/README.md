README
======

```
 Network
 ---------------------------------------------------------------------------------------------------
      |                   |                   |                   |                   |
  +----------------+  +----------------+  +----------------+  +----------------+  +----------------+
  | node01[master] |  | node02[master] |  | node03[slave]  |  | node04[slave]  |  | node05[slave]  |
  | -------------- |  | -------------- |  | -------------- |  | -------------- |  | -------------- |
  | NameNode       |  | SecondNameNode |  | DataNode       |  | DataNode       |  | DataNode       |
  | SecondNameNode |  |                |  | TaskTracker    |  | TaskTracker    |  | TaskTracker    |
  | JobTracker     |  |                |  |                |  |                |  |                |
  |                |  |                |  |                |  |                |  |                |
  +----------------+  +----------------+  +----------------+  +----------------+  +----------------+
```

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

6. Test

   ```
   wget http://norvig.com/big.txt -P ~/
   ./bin/hadoop dfs -copyFromLocal ~/big.txt big.txt
   ./bin/hadoop jar hadoop*examples*.jar wordcount big.txt output
   ./bin/hadoop dfs -ls
   ./bin/hadoop dfs -get output ~/local_output
   ./bin/hadoop dfs -rmr output
   less ~/local_output/part-r-00000
   ```

7. Add a new user

   ```
   sudo useradd user1 -m -s /bin/bash
   hadoop fs -mkdir /user/user1
   hadoop fs -chown user1:user1 /user/user1
   ```

   You should be able to do step 6 by the ``user1`` like this.

   ```
   sudo -i -u user1
   wget http://norvig.com/big.txt -P ~/
   ./bin/hadoop dfs -copyFromLocal ~/big.txt big.txt
   ./bin/hadoop jar hadoop*examples*.jar wordcount big.txt output
   ./bin/hadoop dfs -ls
   ./bin/hadoop dfs -get output ~/local_output
   ./bin/hadoop dfs -rmr output
   less ~/local_output/part-r-00000
   ```

8. Stop Hadoop

   ```
   hdfs@node01$ ./bin/stop-all.sh
   ```
