---
layout: post
title: "Installing hadoop 2.4.0 from scratch (2014 version)"
description: ""
category: Big scale analytics
tags: [Hadoop, Ubuntu, VirtualBox]
---
{% include JB/setup %}
In a [previous post](http://log.malchiodi.com/2013/03/24/installing-hadoop-from-scratch-and-i-mean-it/)
I described how to set up a single-node hadoop cluster in an ubuntu
server running on a virtual machine. Sort story: it's related to my
course on [Big scale analytics](http://malchiodi.di.unimi.it/teaching/big-scale-analytics). Refer to
the original post for the details. As software upgrade is a matter of
fact, I decided to update that tutorial for the more recent 2.4.0
release of hadoop.

I opted for a VM-based solution, so that most of hardware and OS
issues students would face would be limited to installing and
configuring the VM manager. For the records, I am running Mac OS X 10.9.5
and relying on [VirtualBox](https://www.virtualbox.org/) 4.2.8.

First of all, I downloaded the ISO image for Ubuntu server 14.04 at
the [Ubuntu server download page](http://www.ubuntu.com/download/server) and created a Linux-Ubuntu
based VM in VirtualBox with 1GB RAM (who read my previous post will
note an increase in the server's RAM, which is due to the fact that
the default RAM amount of 512MB did lead to hadoop crashes during
simple experiments), a 8GB VDI-based HD (dynamically allocated),
and a DVD preloaded with the Ubuntu server 14.04 ISO image. Then I
ran the VM and followed all default installation options, except for
keyboard layout (I use an italian keyboard). I did not install any
additional software, with the exception of manual package installation
support.

Once the system was up and running, I installed Hadoop following a
mix of the instructions in the tutorials provided by [Michael Noll](http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/),
[BigData Handler](http://bigdatahandler.com/hadoop-hdfs/installing-single-node-hadoop-2.2.0.on-ubuntu/),
and [Rasesh Mori](http://raseshmori.wordpress.com/2012/09/23/install-hadoop-2-0-1-yarn-nextgen/), that is what follows.

Some details about the examples: the host name is `manhattan`, with
an administrator user with login name `boss` (that is, `boss` is a
sudoer); three points (`...`) in a console are used in order to skip
verbose output. Finally, a dollar sign (`$`) occurring at the
beginning of a line denotes the bash prompt.

#### Setting up the environment

First of all, we need to be sure to work on an up-to-date system.
This will probably be the case if the ISO image refers to the current
version of Ubuntu server. Just to be sure, log in as the boss user
and type the following commands.

{% highlight console %}
boss@manhattan:~$ sudo apt-get update
boss@manhattan:~$ sudo apt-get upgrade
{% endhighlight %}

Moreover, it is advisable not to run Hadoop services through a
general-purpose user, so the next step consists in adding a group
`hadoop` and a user `hadoop-user` belonging to that group (for the
purposes of this tutorial, all information requested by `adduser` may
be left blank, except the password.

{% highlight console %}
boss@manhattan:~$ sudo addgroup hadoop
...
boss@manhattan:~$ sudo adduser --ingroup hadoop hadoop-user
...
{% endhighlight %}

#### Installing Java

The mentioned tutorials suggest a potentially unsafe procedure in
order to install the jdk through `apt-get`, thus it's advisable to
opt for a manual installation.

{% highlight console %}
boss@manhattan:~$ wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie gpw_e24=http%3A%2F%2Fwww.oracle.com" "https://edelivery.oracle.com/otn-pub/java/jdk/7u45-b18/jdk-7u45-linux-x64.tar.gz"
...
boss@manhattan:~$ tar -xvzf jdk-7-linux-x64.tar.gz
...
boss@manhattan:~$ sudo mkdir /usr/local/java
boss@manhattan:~$ sudo cp -r jdk1.7.0_45 /usr/local/java
boss@manhattan:~$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.7.0_45/bin/javac" 1
...
boss@manhattan:~$ sudo update-alternatives --set javac /usr/local/java/jdk1.7.0_45/bin/javac
...
{% endhighlight %}

Finally, a couple of environment variables should be set up so that
the java executables are in `$PATH` and hadoop knows where java has
been installed: this is easily accomplished adding

{% highlight bash %}
JAVA_HOME=/usr/local/java/jdk1.7.0_45
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export JAVA_HOME
export PATH
{% endhighlight %}

at the end of `/etc/profile` (to be edited through `su`). When these
variable are in place it is easy to check that java has been properly
installed.

{% highlight console %}
boss@manhattan:~$ . /etc/profile
boss@manhattan:~$ javac -version
javac 1.7.0_45
{% endhighlight %}

#### Setup SSH
All communications with Hadoop are encrypted via SSH, thus the
corresponding server should be installed:

{% highlight console %}
boss@manhattan:~$ sudo apt-get install openssh-server
{% endhighlight %}

and the  `hadoop-user` must be associated to a key pair and
subsequently granting its access to the local machine:

{% highlight console %}
boss@manhattan:~$ su - hadoop-user
hadoop-user@manhattan:~$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
...
The key's randomart image is:
...
hadoop-user@manhattan:~$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
{% endhighlight %}

Now `hadoop-user` should be able to access via ssh to `localhost`
without providing a password:

{% highlight console %}
hadoop-user@manhattan:~$ ssh localhost
The authenticity of host 'localhost (::1)' can't be established.
...
Last login: ...
$
{% endhighlight %}

#### Disable IPV6
Hadoop and IPV6 do not agree on the meaning of `0.0.0.0` address,
thus it is adivsable to disable IPV6 adding the following lines at
the end of `/etc/sysctl.conf` (after having switched back to the
`boss` user):

{% highlight properties %}
# disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
{% endhighlight %}

After a system reboot the output of 
`cat /proc/sys/net/ipv6/conf/all/disable_ipv6` should be `1`, meaning
that IPV6 is actually disabled.

### Hadoop

#### Download and install Hadoop

Download
[hadoop-2.4.0.tar.gz](http://apache.mirrors.pair.com/hadoop/common/hadoop-2.4.0/hadoop-2.4.0.tar.gz),
unpack it and move the results in `/usr/local`, adding a symlink
using the more friendly name `hadoop` and changing ownership of the
directory contents to the `hadoop-user` user:

{% highlight console %}
boss@manhattan:~$ wget wget http://apache.mirrors.pair.com/hadoop/common/hadoop-2.4.0/hadoop-2.4.0.tar.gz
...
boss@manhattan:~$ tar -xzvf hadoop-2.4.0.tar.gz
boss@manhattan:~$ sudo mv hadoop-2.4.0 /usr/local
boss@manhattan:~$ cd /usr/local
boss@manhattan:/usr/local$ sudo ln -s hadoop-2.4.0 hadoop
boss@manhattan:/usr/local$ sudo chown -R hadoop-user:hadoop hadoop-2.4.0
{% endhighlight %}

#### Setup the dedicated user environment

Switch to the `hadoop-user` user and add the following lines at the
end of `~/.bashrc`:

{% highlight sh %}
# Set Hadoop-related environment variables
export HADOOP_PREFIX=/usr/local/hadoop
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
# Native Path
export HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_PREFIX}/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib"
#Java path
export JAVA_HOME='/usr/local/java/jdk1.7.0_45'
# Add Hadoop bin/ directory to PATH
export PATH=$PATH:$HADOOP_HOME/bin:$JAVA_PATH/bin:$HADOOP_HOME/sbin
{% endhighlight %}

In order to have the new environment variables in place, reload
`.bashrc` through `source .bashrc`
get back to the administrator user, then open
`/usr/local/hadoop/etc/hadoop/hadoop-env.sh `, uncomment the line
setting `JAVA_HOME` and set its value to the jdk directory:

{% highlight sh %}
export JAVA_HOME=/usr/local/java/jdk1.7.0_45
{% endhighlight %}

#### Configure Hadoop
Before being able to actually use the hadoop file system it is
necessary to modify some configuration files inside
`/usr/local/hadoop/etc/hadoop`. All such files follow the an XML
format, and the updates should concern the top-level node
`configuration` (likely empty after the hadoop installation).
Specifically:

- in `yarn-site.xml`:
{% highlight xml %}
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
</configuration>
{% endhighlight %}

- in `core-site.xml`:
{% highlight xml %}
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
{% endhighlight %}

- in `mapred-site.xml` (likey to be created through
`cp mapred-site.xml.template mapred-site.xml`):
{% highlight xml %}
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
{% endhighlight %}

- in `hdfs-site.xml`:
{% highlight xml %}
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop/yarn_data/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/usr/local/hadoop/yarn_data/hdfs/datanode</value>
  </property>
</configuration>
{% endhighlight %}

This also requires to manually create the two directories specified
in the `value` XML nodes:
{% highlight console %}
hadoop-user@manhattan:~$ mkdir -p /usr/local/hadoop/yarn_data/hdfs/namenode
hadoop-user@manhattan:~$ mkdir -p /usr/local/hadoop/yarn_data/hdfs/datanode
{% endhighlight %}

#### Formatting the distributed file system
The last step consists in formatting the file system, operation to be
executed as `hadoop-user`:

{% highlight console %}
hadoop-user@manhattan:~$ hdfs namenode -format
...
{% endhighlight %}

the (hopeful) successful result of this operation is specified in one
of the last output lines of previous command.

#### A few more steps and... that's it!
Hadoop is now installed. Invoking the scripts `start-dfs.sh` and
`start-yarn.sh` respectively start the distributed file system and
the mapreduce daemons:

{% highlight console %}
hadoop-user@manhattan:~$ start-dfs.sh
8/10/10 15:28:31 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop-2.4.0/logs/hadoop-hadoop-user-namenode-manhattan.out
localhost: starting datanode, logging to /usr/local/hadoop-2.4.0/logs/hadoop-hadoop-user-datanode-manhattan.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop-2.4.0/logs/hadoop-hadoop-user-secondarynamenode-manhattan.out
8/10/10 15:28:53 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
hadoop-user@manhattan:~$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-user-resourcemanager-manhattan.out
localhost: starting nodemanager, logging to /usr/local/hadoop-2.4.0/logs/yarn-hadoop-user-nodemanager-manhattan.out
{% endhighlight %}

Although it is possible to directly write on the hadoop file system root
directory, it is more advisable to create the user directory for `hadoop-user`,
because all relative paths will refer precisely to this directory:

{% highlight console %}
hadoop-user@manhattan:~$ hdfs dfs -mkdir /user
hadoop-user@manhattan:~$ hdfs dfs -mkdir /user/hadoop-user
{% endhighlight %}

An absence of outputs from these command invokations means a successful
directory creation, which also ensure that the distributed filesystem
component of hadoop has been correctly installed. To test also the mapreduce
component it is possible to run one of the example jobs distributed along with
hadoop:

{% highlight console %}
hadoop-user@manhattan:~$ hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.4.0.jar pi 10 1000
...
Job Finished in 34.136 seconds
Estimated value of Pi is 3.1428571428571
{% endhighlight %}

Finally, to stop the hadoop daemons, simply invoke `stop-dfs.sh` and
`stop-yarn.sh`.