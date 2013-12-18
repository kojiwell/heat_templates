Heat template: Hadoop cluster (5 nodes)
=======================================

Requirements
------------
* Access to a OpenStack Cloud with Heat
* Ubuntu 12.04 LTS cloud image. If you don't have one, you should be able to register the image with this.

  ```
  glance image-create --name <image name> \
      --location http://cloud-images.ubuntu.com/precise/current/precise-server-cloudimg-amd64-disk1.img \
      --disk-format qcow2 \
      --container-format bare \
      --is-public false
  ```

Preferred
---------
* It is better you have your ``~/.ssh/id_dsa.pub`` or ``~/.ssh/id_rsa.pub`` registered. 
  If not, register it like this.

  ```
  nova keypair-add --pub-key ~/.ssh/id_dsa.pub <key name>
  ```

How to
------

1. Create Hadoop cluster

   ```
   heat stack-create hadoop -u https://raw.github.com/kjtanaka/heat_templates/master/hadoop/deploy_hadoop.yml \
       -P "key_name=<key name>;image_name=<image name>"
   ```

   The template creates Hadoop cluster as the diagram below indicates. (IP addresses are differently 
   assigned depending on the environment.)

   ```
    Network
    ---------------------------------------------------------------------------------------------------
         | 192.168.11.1      | 192.168.11.2      | 192.168.11.3      | 192.168.11.4      | 192.168.11.5
     +----------------+  +----------------+  +----------------+  +----------------+  +----------------+
     | node01[master] |  | node02[master] |  | node03[slave]  |  | node04[slave]  |  | node05[slave]  |
     | -------------- |  | -------------- |  | -------------- |  | -------------- |  | -------------- |
     | NameNode       |  | SecondNameNode |  | DataNode       |  | DataNode       |  | DataNode       |
     | SecondNameNode |  |                |  | TaskTracker    |  | TaskTracker    |  | TaskTracker    |
     | JobTracker     |  |                |  |                |  |                |  |                |
     |                |  |                |  |                |  |                |  |                |
     +----------------+  +----------------+  +----------------+  +----------------+  +----------------+
   ```

2. Register hdfs's authorized_keys

   ```
   scp -p hdfs@192.168.11.1:.ssh/authorized_keys hdfs_auth_keys
   scp -p hdfs_auth_keys hdfs@192.168.11.2:.ssh/authorized_keys
   scp -p hdfs_auth_keys hdfs@192.168.11.3:.ssh/authorized_keys
   scp -p hdfs_auth_keys hdfs@192.168.11.4:.ssh/authorized_keys
   scp -p hdfs_auth_keys hdfs@192.168.11.5:.ssh/authorized_keys
   ```

3. Copy /etc/hosts from node01 to the others

   ```
   # Login to node01
   ssh hdfs@192.168.11.1

   # update /etc/hosts
   hdfs@node01:~$ cat /etc/hosts | ssh node02 'sudo sh -c "cat > /etc/hosts"'
     Are you sure you want to continue connecting (yes/no)? yes
   hdfs@node01:~$ cat /etc/hosts | ssh node03 'sudo sh -c "cat > /etc/hosts"'
     Are you sure you want to continue connecting (yes/no)? yes
   hdfs@node01:~$ cat /etc/hosts | ssh node04 'sudo sh -c "cat > /etc/hosts"'
     Are you sure you want to continue connecting (yes/no)? yes
   hdfs@node01:~$ cat /etc/hosts | ssh node05 'sudo sh -c "cat > /etc/hosts"'
     Are you sure you want to continue connecting (yes/no)? yes

   # Register node01(itself) on ~/.ssh/known_hosts.
   hdfs@node01:~$ ssh node01 hostname
     Are you sure you want to continue connecting (yes/no)? yes
   ```

4. Format HDFS

   ```
   hdfs@node01:~$ hadoop namenode -format
   ```

5. Start Hadoop

   ```
   hdfs@node01:~$ ./bin/start-all.sh
   ```

6. Try a WordCount example

   ```
   hdfs@node01:~$ wget http://norvig.com/big.txt -P ~/
   hdfs@node01:~$ hadoop fs -copyFromLocal ~/big.txt big.txt
   hdfs@node01:~$ hadoop jar /usr/lib/hadoop/hadoop*examples*.jar wordcount big.txt output
   hdfs@node01:~$ hadoop fs -ls
   hdfs@node01:~$ hadoop fs -get output ~/local_output
   hdfs@node01:~$ hadoop fs -rmr output
   hdfs@node01:~$ less ~/local_output/part-r-00000
   ```

7. Add a new user, for example, ``john``.

   ```
   hdfs@node01:~$ sudo useradd john -m -s /bin/bash
   hdfs@node01:~$ hadoop fs -mkdir /user/john
   hdfs@node01:~$ hadoop fs -chown john:john /user/john
   ```

   You should be able to try WardCount by ``john`` the same way.

   ```
   hdfs@node01:~$ sudo -i -u john
   john@node01:~$ wget http://norvig.com/big.txt -P ~/
   john@node01:~$ hadoop fs -copyFromLocal ~/big.txt big.txt
   john@node01:~$ hadoop jar /usr/lib/hadoop/hadoop*examples*.jar wordcount big.txt output
   john@node01:~$ hadoop fs -ls
   john@node01:~$ hadoop fs -get output ~/local_output
   john@node01:~$ hadoop fs -rmr output
   john@node01:~$ less ~/local_output/part-r-00000
   ```

8. Stop Hadoop

   ```
   hdfs@node01:~$ ./bin/stop-all.sh
   ```
