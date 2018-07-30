# Setup Hadoop (Standalone)
---
Apache Hadoop terdiri dari beberapa komponen penting yang dapat digunakan untuk menyimpan dan mengolah Big Data. Pada dasarnya Hadoop akan menyimpan dataset dalam HDFS, mengolah pemakaian sumber daya menggunakan YARN, menggunakan mapreduce sebagai pemodelan data berskala besar.

---

## Hardware Instalasi
- VirtualMachine
- 1 Core Processor
- 1GB Memory
- Ubuntu Server 17.10
- 10GB HDD

---

## Kebutuhan Software
- Java 8 JDK
```
apt update && apt upgrade -y
apt install -y openjdk-8-jdk
```
- SSH & PDSH
```
apt install -y ssh pdsh
```

Setelah menjalankan perintah di atas, pastikan tidak muncul log error apapun. Hadoop dan Mapreduce berjaan di environtment java sehingga dibutuhkan jdk. SSH dan PDSH digunakan untuk berkomunikasi dengan cluster yang lainnya ketika Hadoop akan diperbesar skalanya.

---

## Step Instalasi
### 1. Download Hadoop 3.1.0
```
wget http://apache.cs.utah.edu/hadoop/common/hadoop-3.1.0/hadoop-3.1.0.tar.gz
tar -xzvf hadoop-3.1.0.tar.gz
cd hadoop-3.1.0/
```

Pada sistem yang dibuat, digunakan Hadoop versi 3.1.0 dengan mengunduh source-code dari domain apache.cs.utah.edu. Seluruh proses yang dikerjakan pada kegiatan ini dilakukan menggunakan hak akses root. Karena file yang diunduh menggunakan format `tar.gz`, dilakukan dekompresi file menggunakan perintah `tar -zxvf`.

### 2. Sesuaikan konfigurasi Hadoop
    **- etc/hadoop/hdoop-env.sh**
    Environment ini dibutuhkan agar Hadoop mengenali lokasi jdk.
    ```
    export JAVA_HOME=/usr
    ```
    **- etc/hadoop/core-site.xml**
    Pada konfigurasi ini Hadoop akan berjalan pada port 9000.
    ```
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://localhost:9000</value>
        </property>
    </configuration>
    ```
    **- etc/hadoop/hdfs-site.xml**
    Pada konfigurasi ini Hadoop akan menjalankan 1 replika.
    ```
    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>1</value>
        </property>
    </configuration>
    ```

### 3. Buat ssh key pairs
```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
echo "ssh" > /etc/pdsh/rcmd_default
```
SSH keypairs digunakan untuk membuka soket ssh sehingga hadoop dapat berjalan dengan baik. Penggunaan keypairs adalah agar ssh bisa berjalan tanpa memasukkan autentikasi standar (username/password). Tidak ada perbedaan generate ssh keypair pada Hadoop dengan generate ssh keypair pada umumnya.

### 4. Pasang environtment Hadoop pada ~/.bashrc
```
export HDFS_NAMENODE_USER="root"
export HDFS_DATANODE_USER="root"
export HDFS_SECONDARYNAMENODE_USER="root"
export YARN_RESOURCEMANAGER_USER="root"
export YARN_NODEMANAGER_USER="root"
```
Pada tahap ini dilakukan pengaturan environment HDFS dan YARN untuk NameNode, DataNode, ResourceManager, dan NODEMANAGER. Nantinya environment tersebut akan digunakan pada file konfigurasi site yarn-site.

### 5. Format HDFS filesystem dan jalankan HDFS
```
bin/hdfs namenode -format
sbin/start-dfs.sh
```
![alt text](https://github.com/nauticas/hadoop-standalone/blob/master/images/HDFS-Web_Interface_1.jpg "Output 1 HDFS Web Interface")
![alt text](https://github.com/nauticas/hadoop-standalone/blob/master/images/HDFS-Web_Interface_2.jpg "Output 2 HDFS Web Interface")

### 6. Sesuaikan konfigurasi YARN
    **- Buat direktori kerja YARN**
    ```
    bin/hdfs dfs -mkdir /user
    bin/hdfs dfs -mkdir /user/root
    ```
    **- etc/hadoop/mapred-site.xml**
    ```
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
    ```
    **- etc/hadoop/yarn-site.xml**
    ```
    <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
    </property>
    <property>
            <name>yarn.nodemanager.vmem-check-enabled</name>
            <value>false</value>
    </property>
    <property>
            <name>yarn.app.mapreduce.am.env</name>
            <value>HADOOP_MAPRED_HOME=/root/hadoop-3.1.0</value>
    </property>
    <property>
            <name>mapreduce.map.env</name>
            <value>HADOOP_MAPRED_HOME=/root/hadoop-3.1.0</value>
    </property>
    <property>
            <name>mapreduce.reduce.env</name>
            <value>HADOOP_MAPRED_HOME=/root/hadoop-3.1.0</value>
    </property>
    ```

### 7. Jalankan YARN
```
sbin/start-yarn.sh
```
![alt text](https://github.com/nauticas/hadoop-standalone/blob/master/images/Yarn-Web_Interface.jpg "Output YARN Web Interface")

---

## Lakukan Uji Coba Hadoop
Pengujian Hadoop dilakukan dengan menjalankan perintah berikut:
```
bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.0.jar pi 16 1000
```

**Output:**
```
Job Finished in 240.177 seconds
Estimated value of Pi is 3.14250000000000000000
```
![alt text](https://github.com/nauticas/hadoop-standalone/blob/master/images/Yarn-Test_Output_1.jpg "Output 1 Test YARN")
![alt text](https://github.com/nauticas/hadoop-standalone/blob/master/images/Yarn-Test_Output_2.jpg "Output 2 Test YARN")

Menggunakan `yarn`, dilakukan penghitungan nilai `PI` sampai nilai desimal `16` menggunakan metode `quasiMonteCarlo`. Pengujian berlangsung selama sekitar 240 detik pada sistem yang sudah dibuat.

