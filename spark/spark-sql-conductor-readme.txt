1: set mandatory env
export CLUSTERADMIN=sym
export CLUSTERNAME=cluster123
export BASEPORT=17869

2: run bin installation, usually set NFS or GPFS shared folder to install
[root@conduct01 con]# ./conductorspark2.1.0.0_x86_64.bin --prefix /opt/sym --dbpath /opt/sym/DB
The following commands will install the .rpm packages on the host.
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB egocore-3.4.0.0.x86_64.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB egojre-1.8.0.3.x86_64.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB egowlp-8.5.5.9.noarch.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB egorest-3.4.0.0.noarch.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB egomgmt-3.4.0.0.noarch.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB egoelastic-1.0.0.0.x86_64.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB egogpfsmonitor-3.4.0.0.noarch.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB ascd-2.1.0.0.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB conductorsparkcore-2.1.0.0.rpm
rpm -ivh --prefix /opt/sym --dbpath /opt/sym/DB conductorsparkmgmt-2.1.0.0.rpm

3: egoconfig master and different nodes
On Master:
su - sym & . profile.platform
egoconfig join conduct01
egoconfig entitlement entitlement.file
su - root & . profile.platform & egosh ego start
On Nodes
su - sym & . profile.platform & egoconfig join conduct01
su - root & . profile.platform & egosh ego start

4: open below URL to get certificate
https://conduct01.eng.platformlab.ibm.com:8443/platform
https://conduct01.eng.platformlab.ibm.com:8543/platform
https://conduct01.eng.platformlab.ibm.com:8643/platform
https://conduct01.eng.platformlab.ibm.com:5601    this is Kibana URL

5: Deploy Spark
create at least 3 Resource Group:  driven, executor and shuffle
create Consumer: spark01 (involve above 3 RG)

6: run a Pi Spark example: Workload-->Spark-->Applications & Notebooks-->Run or Schedule a batch
--deploy-mode cluster --class org.apache.spark.examples.SparkPi /opt/aaa/spark-1.6.1-hadoop-2.6/lib/spark-examples-1.6.1-hadoop2.6.0.jar 

SPARK SQL examples
1. Install MySQL as database server, assume on conduct01 
2: download hive.apache.org and set all ENV
export JAVA_HOME=/opt/Linux_jdk1.7.0_x86_64
export HADOOP_HOME=/opt/hadoop-2.6.4
#export HIVE_HOME=/opt/hive-2.0.1-bin
export HIVE_HOME=/opt/hive-2.1.0-bin
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HIVE_HOME/bin
export CLASSPATH=$CLASSPATH:$HIVE_HOME/lib

set hive-env.sh
HADOOP_HOME=/opt/hadoop-2.6.4
export HIVE_CONF_DIR=/opt/hive-2.1.0-bin/conf
export HIVE_AUX_JARS_PATH=/opt/hive-2.1.0-bin/lib
export JAVA_HOME=/opt/Linux_jdk1.7.0_x86_64

3: set hive-site.xml
Find hive-default.xml.template from Hive configuration directory, copy to $SPARK_HOME/conf and rename to hive-site.xml. Change below parameters based on your environment. I attached a hive-site.xml in the mail.

    javax.jdo.option.ConnectionURL         
        jdbc:mysql://192.168.1.93:3306/sparkmetadata?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8
    javax.jdo.option.ConnectionDriverName
        com.mysql.jdbc.Driver
    javax.jdo.option.ConnectionUserName
        root
    javax.jdo.option.ConnectionPassword
        xxx

Optional, if you found ${system:java.io.tmpdir} related error message in output when you run spark-sql command, you can change below settings to local directory in hive-site.xml file
    hive.exec.local.scratchdir
    hive.downloaded.resources.dir

4. Copy mysql-connector-java-5.1.39-bin.jar to $SPARK_HOME/lib
   Add mysql-connector-java-5.1.39-bin.jar into Zeppelin CLASSPATH according to previous mail on all hosts.
5. Restart SIG

6: run SPARK SQL and check
/opt/ccc/spark-1.6.1-hadoop-2.6/bin/spark-sql --deploy-mode client --master spark://conduct01.eng.platformlab.ibm.com:7079 --conf spark.ego.uname=Admin --conf spark.ego.passwd=Admin --driver-class-path /opt/mysql-connector-java-5.1.39-bin.jar

spark-sql> CREATE TABLE vertices(ID BigInt,Title String) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LINES TERMINATED BY '\n';
spark-sql> show tables;
mysql> select * from TBLS;


Run spark to submit a job
$SPARK_HOME/bin/spark-submit --master spark://conduct01.eng.platformlab.ibm.com:7079 --deploy-mode cluster --class org.apache.spark.examples.SparkPi /opt/ccc/spark-1.6.1-hadoop-2.6/lib/spark-examples-1.6.1-hadoop2.6.0.jar
