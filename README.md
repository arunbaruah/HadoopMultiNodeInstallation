# HadoopMultiNodeInstallation
This repo contains only one MD file having the installation script for cut/copy paste.
Please find the below instructions and commands to install Cloudera CDH5 (YARN) Hadoop in CentOS in any cloud environment. I am using IBM
Softlayer cloud server. Here I have used total 4 systems. One Master and three Slave nodes.

HADOOP 2.X (YARN) INSTALLATION IN MULTI CLUSTER - Here I will prepare a four node cluster in IBM Softlayer Cloud Servers

Follow the process stated below:

1) Install Centos 6.x (minimal installation) in local system or in cloud servers

2) Install Java 1.7x (minimum) or higher
    a) 
    cd /opt/

    b)
    wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.tar.gz"

    c)
    tar xzf jdk-7u79-linux-x64.tar.gz

    d)Install Java with Alternatives
    cd /opt/jdk1.7.0_79/ 

    alternatives --install /usr/bin/java java /opt/jdk1.7.0_79/bin/java 2 

    alternatives --config java

    e)Now you may also required to set up javac and jar commands path using alternatives command.

    alternatives --install /usr/bin/jar jar /opt/jdk1.7.0_79/bin/jar 2 

    alternatives --install /usr/bin/javac javac /opt/jdk1.7.0_79/bin/javac 2 

    alternatives --set jar /opt/jdk1.7.0_79/bin/jar 

    alternatives --set javac /opt/jdk1.7.0_79/bin/javac

    e) Check Installed Java Version

    java -version  

    f) Configuring Environment Variables - add following commands to any start-up script like ~/.bashrc or ~/.bash_profile.

    export JAVA_HOME=/opt/jdk1.7.0_79

    export JRE_HOME=/opt/jdk1.7.0_79/jre

    export PATH=$PATH:/opt/jdk1.7.0_79/bin:/opt/jdk1.7.0_79/jre/bin

    g) source ~/.bashrc  -- to refresh or reload the env. vars

    ---------------------------- Java part completed ---------------------------


3) Just install Hadoop in all the systems. We will configure later and set necessary configurations inc ssh keygen etc.

	a) Here we would be using Cloudera CDH5 -- download it

	wget http://archive.cloudera.com/cdh5/one-click-install/redhat/6/x86_64/cloudera-cdh-5-0.x86_64.rpm

	b) Install cdh5 repo downloaded in above step in All Nodes (all machines)

	yum --nogpgcheck localinstall cloudera-cdh-5-0.x86_64.rpm

	c) Once installation is complete follow the below steps from 4

4) When installation is successful in all the system then follow the configuration process.

   a) Add hostname and ip information to /etc/hosts file on each host. (provide private ip for all other nodes for internal comm)

   vi /etc/hosts 

   	127.0.0.1 localhost.localdomain localhost
	50.97.40.222 hadoopmaster.qualonix.softlayer.com hadoopmaster.qualonix
	10.56.64.157 hadoopslave1.qualonix.softlayer.com hadoopslave1.qualonix
	10.56.64.147 hadoopslave2.qualonix.softlayer.com hadoopslave2.qualonix
	10.56.64.131 hadoopslave3.qualonix.softlayer.com hadoopslave3.qualonix

   ---------------------------------------------------------------------------

    127.0.0.1 localhost.localdomain localhost
	50.97.40.221 hadoopslave1.qualonix.softlayer.com hadoopslave1.qualonix
	10.56.64.159 hadoopmaster.qualonix.softlayer.com hadoopmaster.qualonix
	10.56.64.147 hadoopslave2.qualonix.softlayer.com hadoopslave2.qualonix
	10.56.64.131 hadoopslave3.qualonix.softlayer.com hadoopslave3.qualonix

  ---------------------------------------------------------------------------

    127.0.0.1 localhost.localdomain localhost
	50.97.40.220 hadoopslave2.qualonix.softlayer.com hadoopslave2.qualonix
	10.56.64.157 hadoopslave1.qualonix.softlayer.com hadoopslave1.qualonix
	10.56.64.159 hadoopmaster.qualonix.softlayer.com hadoopmaster.qualonix
	10.56.64.131 hadoopslave3.qualonix.softlayer.com hadoopslave3.qualonix

---------------------------------------------------------------------------

	127.0.0.1 localhost.localdomain localhost
	50.97.40.219 hadoopslave3.qualonix.softlayer.com hadoopslave3.qualonix
	10.56.64.147 hadoopslave2.qualonix.softlayer.com hadoopslave2.qualonix
	10.56.64.157 hadoopslave1.qualonix.softlayer.com hadoopslave1.qualonix
	10.56.64.159 hadoopmaster.qualonix.softlayer.com hadoopmaster.qualonix

	b) ping hadoopslave3.qualonix.softlayer.com (ping all system)

    c) Please stop the firewall and disable the selinux.

       To stop firewall in centos : 

       service iptables stop && chkconfig iptables off

       To disable selinux :

       vi /etc/selinux/config

       once file is opened, please verify that “SELINUX=disabled” is set.

    d) Date should be in sync 

       date

    f) We need passwordless configuration so follow the below steps:

       a) Generate rsa key pair using ssh-keygen command

       ssh-keygen

       b) Copy generated public key to the slave nodes

       ssh-copy-id -i ~/.ssh/id_rsa.pub root@hadoopslave1.qualonix.softlayer.com

       ssh-copy-id -i ~/.ssh/id_rsa.pub root@hadoopslave3.qualonix.softlayer.com

       ssh-copy-id -i ~/.ssh/id_rsa.pub root@hadoopslave3.qualonix.softlayer.com

       Now try logging into the machine, with “ssh slaves”

       root@hadoopslave1.qualonix.softlayer.com
       exit --- and test for all

       -----------------  successfully configured passwordless ssh between master and slave node --------------------

    5. House keeping job is completed now it's time to Install and deploy ZooKeeper and prepare Hadoop cluster. We will do the following:

       a) In Hadoop Master -- we will install the following:
          i)	zookeeper
          ii)	Namenode
          iii)	resource manager (YARN)
          iv)	history server and yarn proxyserver (Optional)

       b) In Slave#1 we will install the secondarynamenode. In production env, this will be another machine, but for me I am using slave1
          i)	secondarynamenode
          ii)   nodemanager
          iii)  datanode
          iv)   mapreduce

        Now let us complete one by one.

        In master - 

        1)
        yum -y install zookeeper-server

        2)  create zookeeper dir and apply permissions
        
        mkdir -p /var/lib/zookeeper
        chown -R zookeeper /var/lib/zookeeper/

        Init zookeeper and start the service

        service zookeeper-server init

        service zookeeper-server start

        3)  Install namenode

        yum -y install hadoop-hdfs-namenode

        4)   resource manager

        yum -y install hadoop-yarn-resourcemanager

        5)   history server and yarn proxyserver

        yum -y install hadoop-mapreduce-historyserver hadoop-yarn-proxyserver

        6)  yum -y install hadoop-client

        In Slave#1 -- Install secondary namenode and remaining 

        1) 
        yum -y install hadoop-hdfs-secondarynamenode

        2) nodemanager, datanode & mapreduce and client pkg

        yum -y install hadoop-yarn-nodemanager hadoop-hdfs-datanode hadoop-mapreduce

        yum -y install hadoop-client

        7) In Slave#2 and slave#3 we will install nodemanager, datanode & mapreduce

         yum -y install hadoop-yarn-nodemanager hadoop-hdfs-datanode hadoop-mapreduce

         yum -y install hadoop-client

-----------------------Completed installation now it's time to deploy HDFS & YARN------------------
        
    6. Executes the below commands for all the machines one by one

    a)

    cp -r /etc/hadoop/conf.empty /etc/hadoop/conf.my_cluster

    alternatives --install /etc/hadoop/conf hadoop-conf /etc/hadoop/conf.my_cluster 50

    alternatives --set hadoop-conf /etc/hadoop/conf.my_cluster

    b) configure hdfs properties --  /etc/hadoop/conf/ dir on master node and edit the below properties

    vi /etc/hadoop/conf/core-site.xml

    Add below lines in it under <configuration> tag

    	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://master.hadoop.com:8020</value>
		</property>

		--------------------------------------------

    vi /etc/hadoop/conf/hdfs-site.xml

	    <property>
		<name>dfs.permissions.superusergroup</name>
		<value>hadoop</value>
		</property>
		<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///data/1/dfs/nn,file:///nfsmount/dfs/nn</value>
		</property>
		<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///data/1/dfs/dn,file:///data/2/dfs/dn,file:///data/3/dfs/dn,file:///data/4/dfs/dn</value></property>
		<property>
		<name>dfs.namenode.http-address</name>
		<value>192.168.111.130:50070</value>
		<description>
		The address and the base port on which the dfs NameNode Web UI will listen.
		</description>
		</property>

        -------------------------------------------

     Now we will copy the above two files in all the slave nodes

        scp core-site.xml hdfs-site.xml slave.hadoop.com:/etc/hadoop/conf/

    c) Create local directories

       on master ---

        mkdir -p /data/1/dfs/nn /nfsmount/dfs/nn
		chown -R hdfs:hdfs /data/1/dfs/nn /nfsmount/dfs/nn
		chmod 700 /data/1/dfs/nn /nfsmount/dfs/nn
		chmod go-rx /data/1/dfs/nn /nfsmount/dfs/nn

	   on slave ---

	    mkdir -p /data/1/dfs/dn /data/2/dfs/dn /data/3/dfs/dn /data/4/dfs/dn
		chown -R hdfs:hdfs /data/1/dfs/dn /data/2/dfs/dn /data/3/dfs/dn /data/4/dfs/dn

	d) Format the namenode

	    sudo -u hdfs hdfs namenode -format

	f)  Start hdfs services -- master and slave

	   for x in `cd /etc/init.d ; ls hadoop-hdfs-*` ; do service $x start ; done

	g) Create hdfs tmp dir on any of the hadoop node, we will do it in slave#1

	   sudo -u hdfs hadoop fs -mkdir /tmp
	   sudo -u hdfs hadoop fs -chmod -R 1777 /tmp

    -------------------------- HDFS Completed ------------------------------------

    Finally Deploy YARN

    a) Prepare yarn configuration properties
       replace your /etc/hadoop/conf/mapred-site.xml with below contents on master host

       [root@master conf]# cat mapred-site.xml

        <configuration>
		<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
		</property>
		<property>
		<name>yarn.app.mapreduce.am.staging-dir</name>
		<value>/user</value>
		</property>
		</configuration>

      ---------------------------------------------------------------------------

      b) Replace your /etc/hadoop/conf/yarn-site.xml with below contents on master host
 
      [root@master conf]# cat yarn-site.xml

      <configuration>
		<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
		</property>
		<property>
		<name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
		<value>org.apache.hadoop.mapred.ShuffleHandler</value>
		</property>
		<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
		</property>
		<property>
		<description>List of directories to store localized files in.</description>
		<name>yarn.nodemanager.local-dirs</name>
		<value>file:///var/lib/hadoop-yarn/cache/${user.name}/nm-local-dir</value>
		</property>
		<property>
		<description>Where to store container logs.</description>
		<name>yarn.nodemanager.log-dirs</name>
		<value>file:///var/log/hadoop-yarn/containers</value>
		</property>
		<property>
		<description>Where to aggregate logs to.</description>
		<name>yarn.nodemanager.remote-app-log-dir</name>
		<value>hdfs://master.hadoop.com:8020/var/log/hadoop-yarn/apps</value>
		</property>
		<property>
		<description>Classpath for typical applications.</description>
		<name>yarn.application.classpath</name>
		<value>
		$HADOOP_CONF_DIR,
		$HADOOP_COMMON_HOME/*,$HADOOP_COMMON_HOME/lib/*,
		$HADOOP_HDFS_HOME/*,$HADOOP_HDFS_HOME/lib/*,
		$HADOOP_MAPRED_HOME/*,$HADOOP_MAPRED_HOME/lib/*,
		$HADOOP_YARN_HOME/*,$HADOOP_YARN_HOME/lib/*
		</value>
		</property>
		<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>slave.hadoop.com</value>
		</property>
		<property>
		<name>yarn.nodemanager.local-dirs</name>
		<value>file:///data/1/yarn/local,file:///data/2/yarn/local,file:///data/3/yarn/local</value>
		</property>
		<property>
		<name>yarn.nodemanager.log-dirs</name>
		<value>file:///data/1/yarn/logs,file:///data/2/yarn/logs,file:///data/3/yarn/logs</value>
		</property>
		</configuration>



		---------------------------------------------------------------------------------------------


		c)  Now Copy modified files to slave machines.

        scp mapred-site.xml yarn-site.xml slave.hadoop.com:/etc/hadoop/conf/

        d) Configure local directories for yarn in master machine

        mkdir -p /data/1/yarn/local /data/2/yarn/local /data/3/yarn/local /data/4/yarn/local
        mkdir -p /data/1/yarn/logs /data/2/yarn/logs /data/3/yarn/logs /data/4/yarn/logs
        chown -R yarn:yarn /data/1/yarn/local /data/2/yarn/local /data/3/yarn/local /data/4/yarn/local
        chown -R yarn:yarn /data/1/yarn/logs /data/2/yarn/logs /data/3/yarn/logs /data/4/yarn/logs

        e) Configure the history server in master -- mapred-site.xml ( /etc/hadoop/conf/mapred-site.xml )

        <property>
		<name>mapreduce.jobhistory.address</name>
		<value>master.hadoop.com:10020</value>
		</property>
		<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>master.hadoop.com:19888</value>
		</property>

        ---------------------------------------------------------------------------------------------

        f) Configure proxy settings for history server

        Add below properties in /etc/hadoop/conf/core-site.xml

        <property>
		<name>hadoop.proxyuser.mapred.groups</name>
		<value>*</value>
		</property>
		<property>
		<name>hadoop.proxyuser.mapred.hosts</name>
		<value>*</value>
		</property>

     ---------------------------------------------------------------------------------------------

     g) Copy modified files to slave machines

     scp mapred-site.xml core-site.xml slave.hadoop.com:/etc/hadoop/conf/

     h) Create history directories and set permissions -- in master

     sudo -u hdfs hadoop fs -mkdir -p /user/history
     sudo -u hdfs hadoop fs -chmod -R 1777 /user/history
     sudo -u hdfs hadoop fs -chown mapred:hadoop /user/history

     i) Create log directories and set permissions -- master

      sudo -u hdfs hadoop fs -mkdir -p /var/log/hadoop-yarn
      sudo -u hdfs hadoop fs -chown yarn:mapred /var/log/hadoop-yarn


     j) Verify hdfs file structure -- master

      sudo -u hdfs hadoop fs -ls -R /

     k) Start yarn and Jobhistory server -- master

     sudo service hadoop-yarn-resourcemanager start

     l) In slaves

     sudo service hadoop-yarn-nodemanager start

     m) Start history server on master

     sudo service hadoop-mapreduce-historyserver start

     n)  Create user for running mapreduce jobs - master

     sudo -u hdfs hadoop fs -mkdir /user/arun
     sudo -u hdfs hadoop fs -chown kuldeep /user/arun

     o) set core hadoop services to auto start when OS boot ups

     Master --

     sudo chkconfig hadoop-yarn-resourcemanager on
     sudo chkconfig hadoop-hdfs-namenode on
     sudo chkconfig hadoop-mapreduce-historyserver on

     On all slaves

     slave#1 has one extra i.e. secNameNode

     sudo chkconfig hadoop-hdfs-secondarynamenode on

     sudo chkconfig hadoop-yarn-nodemanager on
     sudo chkconfig hadoop-hdfs-datanode on

     --------------------------- All Set, Now let's check in browser -------------------------

     Use master's public IP address

     NameNode UI
     50.97.40.222:50070/

     YARN UI
     50.97.40.222:8080/cluster

     history server
     50.97.40.222:19888/jobhistory

     Secondary Namenode
     50.97.40.221:50090/status.html


     
     ------------------------------ Our installation is completed with this ---------------------

     You are free to use this file and it's available in my github account

     























