---
layout: post
title: "Install Hadoop YARN 2.2.0 on MacOS X Mavericks"
description: ""
category: 
tags: []
---
{% include JB/setup %}

This post is a copy of [this post](http://ac31004.blogspot.com/2013/10/installing-hadoop-2-on-mac_29.html) on Blogspot.com.

Here are the steps I used to get Hadoop 2.2.0 working on my MAC running OSX 10.9


Get hadoop from http://www.apache.org/dyn/closer.cgi/hadoop/common/

make sure JAVA_HOME is set (if you have Java 6 on your machine):

    export JAVA_HOME=`/usr/libexec/java_home -v1.6`

point HADOOP_INSTALL to the hadoop installation directory

    export HADOOP_INSTALL=/Applications/hadoop-2.2.0

And set the path

    export PATH=$PATH:$HADOOP_INSTALL/bin:$HADOOP_INSTALL/sbin

You can test hadoop is found with

    hadoop version

Make sure ssh is set up on your machine by ticking `System Preferences -> Sharing -> Remote Login`.  And verify it by :

    ssh localhost

where is the name you used to logon.

In `$HADOOP_INSTALL/etc` these are the conf files I changed.

`core-site.xml`

     <configuration>  
       <property>  
         <name>fs.default.name</name>  
           <value>hdfs://localhost:9000</value>  
       </property>  
     </configuration>  

`hdfs-site.xml`

     <configuration>  
       <property>  
         <name>dfs.replication</name>  
         <value>1</value>  
       </property>  
       <property>  
         <name>dfs.namenode.name.dir</name>  
         <value>file:/Users/Administrator/hadoop/namenode</value>  
       </property>  
       <property>  
         <name>dfs.datanode.data.dir</name>  
         <value>file:/Users/Administrator/hadoop/datanode</value>  
       </property>  
    </configuration>  

Make the directories for the namenode and datanode data (note the file above and the mkdir below will need to reflect where you  want to store the files, I've stored mine in the home directory of the Administrator user on my Mac).

    mkdir -p /Users/Administrator/hadoop/namenode
    mkdir -p /Users/Administrator/hadoop/datanode

    hadoop namenode -format

`yarn-site.xml`

    <configuration>  
      <!-- Site specific YARN configuration properties -->  
      <property>  
        <name>yarn.resourcemanager.address</name>  
        <value>localhost:8032</value>  
      </property>  
      <property>  
        <name>yarn.nodemanager-aux-services</name>  
        <value>madpreduce.shuffle</value>  
      </property>  
    </configuration>  


    start-dfs.sh
    start-yarn.sh

    jps

`jps` should list the following Java processes:

    9430 ResourceManager
    9325 SecondaryNameNode
    9513 NodeManager
    9225 DataNode
    9916 Jps
    9140 NameNode

If not check log files.  If data node note started and  you get incompatible id's error, stop everything delete datanode directory and recreate
datanode directory

Try accessing the DFS:

    hadoop fs -ls

If you get 

    ls: `.': No such file or directory

then there is no home directory in the hadoop file system.  So

    hadoop fs -mkdir /user
    hadoop fs -mkdir /user/<username>

where is the name you are logged onto the machine with.

Now change to `$HADOOP_INSTALL` directory and upload a file

    cd $HADOOP_INSTALL
    hadoop fs -put LICENSE.txt 


Finally try a mapreduce job:

    cd share/hadoop/mapreduce
    hadoop jar ./hadoop-mapreduce-examples-2.2.0 wordcount LICENSE.txt out