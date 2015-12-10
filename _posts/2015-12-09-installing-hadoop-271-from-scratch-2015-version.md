---
layout: post
title: "Installing hadoop 2.7.1 from scratch (2015 version)"
description: ""
category: Big scale analytics
tags: [Hadoop, Ubuntu, VirtualBox]
---
{% include JB/setup %}
In two previous posts I described the installation process for
the [2.4.0](http://log.malchiodi.com/2014/10/09/installing-hadoop-240-from-scratch-2014-version) and the [0.20](http://log.malchiodi.com/2013/03/24/installing-hadoop-from-scratch-and-i-mean-it/) releases of hadoop to the students of my class on on [Big scale analytics](http://malchiodi.di.unimi.it/teaching/big-scale-analytics).

I opted for a VM-based solution, so that most of hardware and OS
issues students would face would be limited to installing and
configuring the VM manager. For the records, I am running Mac OS X 10.10.5
and relying on [VirtualBox](https://www.virtualbox.org/) 5.0.10.

First of all, I downloaded the ISO image for Ubuntu server 14.04 at
the [Ubuntu server download page](http://www.ubuntu.com/download/server) and created a Linux-Ubuntu
based VM in VirtualBox with 1GB RAM, a 8GB VDI-based HD (dynamically allocated),
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

The next step consists in installing Java 8, for instance through the
[Webupd8 repository](http://www.webupd8.org/):

{% highlight console %}
boss@manhattan:~$ sudo apt purge openjdk*
...
boss@manhattan:~$ sudo add-apt-repository -y ppa:webupd8team/java
...
boss@manhattan:~$ sudo apt update
...
boss@manhattan:~$ sudo apt install -y oracle-java8-installer
...
{% endhighlight %}

After having accepted the software license, we can check that the correct
release has been installed:

{% highlight console %}
boss@manhattan:~$ java -version
java version "1.8.0_66"
...
{% endhighlight %}

Finallly, the `$JAVA_HOME` environment variable should be properly set up:

{% highlight bash %}
boss@manhattan:~$ sudo sh -c 'echo "export JAVA_HOME=/usr" >> /etc/profile'
boss@manhattan:~$ source /etc/profile
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

#### Setup SSH
All communications with Hadoop are encrypted via SSH, thus the
corresponding server should be installed:

{% highlight console %}
boss@manhattan:~$ sudo apt-get install openssh-server
{% endhighlight %}

and the `hadoop-user` must be associated to a key pair and
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

### Hadoop

#### Download and install Hadoop

Download
[hadoop-2.7.1.tar.gz]( http://mirror.nohup.it/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz) (the link points to a suggested apache mirror, thus feel free to change it
into a nearest link),
unpack it and move the results in `/usr/local`, adding a symlink
using the more friendly name `hadoop` and changing ownership of the
directory contents to the `hadoop-user` user:

{% highlight console %}
boss@manhattan:~$ wget http://mirror.nohup.it/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz
...
boss@manhattan:~$ tar xzf hadoop-2.7.1.tar.gz
boss@manhattan:~$ rm hadoop-2.7.1.tar.gz
boss@manhattan:~$ sudo mv hadoop-2.7.1 /usr/local
boss@manhattan:~$ sudo ln -sf /usr/local/hadoop-2.7.1/ /usr/local/hadoop
boss@manhattan:~$ sudo chown -R hadoop-user:hadoop /usr/localhadoop-2.7.1/
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
# Native path
export HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_PREFIX}/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib/native"
# Java path
export JAVA_HOME="/usr"
# Add Hadoop bin/ directory to PATH
export PATH=$PATH:$HADOOP_HOME/bin:$JAVA_PATH/bin:$HADOOP_HOME/sbin
{% endhighlight %}

In order to have the new environment variables in place, reload
`.bashrc`:

{% highlight sh %}
hadoop-user@manhattan:~$ source .bashrc
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
</configuration>
{% endhighlight %}

- in `core-site.xml`:
{% highlight xml %}
<configuration>
  <property>
	<name>fs.defaultFS</name>
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
</configuration>
{% endhighlight %}

Finally, set to `/usr` the `JAVA_HOME` variable in `/usr/local/hadoop/etc/hadoop/hadoop-env.sh`.

#### Formatting the distributed file system
The last step consists in formatting the file system, operation to be
executed as `hadoop-user`:

{% highlight console %}
hadoop-user@manhattan:~$ hdfs namenode -format
...
{% endhighlight %}

the (hopeful) successful result of this operation is specified within the
(quite verbose) output: search for the text `successfully formatted`!

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