---
layout: post
title: "Installing hadoop from scratch. And I mean it"
description: ""
category: 
tags: [Hadoop, Ubuntu]
---
{% include JB/setup %}
This year I started a new course on [Big scale
analytics](http://malchiodi.di.unimi.it/teaching/big-scale-analytics),
and the first tool I introduced students to is, not surprisingly,
[Hadoop](http://hadoop.apache.org/). Having pointed out that this
tool is useful when run on a significant cluster, I nonetheless wanted
students to be able to install their own copy of the software even
on a single-node cluster. Maybe some among them will have sometime to run
a cluster on their own. Moreover (and most importantly), this will let
them to autonomously run a job, experiment and debug on small scale before
sending everything, for instance, to [AWS](http://aws.amazon.com).

I opted for a VM-based solution, so that most of hardware and OS issues
students would face would be limited to installing and configuring the
VM manager. For the records, I am running Mac OS X 10.7.5
and relying on [VirtualBox](https://www.virtualbox.org/) 4.2.8. The
rest of this post documents the steps I followed to get a single-node
Hadoop cluster running.

First of all, I downloaded the ISO image for Ubuntu server 12.10 at the
[Ubuntu server download page](http://www.ubuntu.com/download/server) and
created a Linux-Ubuntu based VM in VirtualBox with default settings (that
is, 512 MB of RAM and a 8GB VDI-based HD, dynamically allocated) and a DVD
preloaded with the Ubuntu server 12.10 ISO image. Then I ran the VM and
followed all default installation options, except for keyboard layout (I
use an italian keyboard). I did not install any additional software, with the
exception of manual package installation support.

Once the system was up and running, I installed Hadoop (almost) following
the instructions in the [tutorial written by Michael
Noll](http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/),
that is what follows.

Some details about the examples: the host name is `manhattan`, with an
administrator user with login name `boss`; three points (`...`) in a console
are used in order to skip verbose output.

#### Installing Java

The mentioned tutorial suggest a potentially unsafe procedure in order to
install the jdk through `apt-get`, thus I opted for a manual installation:

{% highlight console %}
boss@manhattan:~$ wget --no-cookies --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com" "http://download.oracle.com/otn-pub/java/jdk/7/jdk-7-linux-x64.tar.gz"
...
boss@manhattan:~$ tar -xvf jdk-7-linux-x64.tar.gz
...
boss@manhattan:~$ sudo mkdir -p /usr/lib/jvm/jdk1.7.0 
boss@manhattan:~$ sudo mv jdk1.7.0_03/* /usr/lib/jvm/jdk1.7.0/
boss@manhattan:~$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.7.0/bin/java" 1
...
boss@manhattan:~$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.7.0/bin/javac" 1
...
boss@manhattan:~$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.7.0/bin/javaws" 1
...
boss@manhattan:~$ javac -version
javac 1.7.0
{% endhighlight %}

#### Adding a dedicated user
It is advisable not to run Hadoop services through a general-purpose user,
so the next step consists in adding a group `hadoop` and a user `hduser`
belonging to that group.

{% highlight console %}
boss@manhattan:~$ sudo addgroup hadoop
...
boss@manhattan:~$ sudo adduser --ingroup hadoop hduser
...
{% endhighlight %}

#### Setup SSH
All communications with Hadoop are encrypted via SSH, thus the corresponding
server should be installed:

{% highlight console %}
boss@manhattan:~$ sudo apt-get install openssh-server
{% endhighlight %}

and the  `hduser` must be associated to a key pair and subsequently granting
its access to the local machine:

{% highlight console %}
boss@manhattan:~$ su - hduser
hduser@manhattan:~$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
...
The key's randomart image is:
...
hduser@manhattan:~$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
{% endhighlight %}

Now the `hduser` should be able to access via ssh to `localhost`:

{% highlight console %}
hduser@manhattan:~$ ssh localhost
The authenticity of host 'localhost (::1)' can't be established.
...
Last login: ...
$
{% endhighlight %}

#### Disable IPV6
Hadoop and IPV6 do not agree on the meaning of `0.0.0.0` address, thus it is
adivsable to disable IPV6 adding the following lines at the end of
`/etc/sysctl.conf` (after having switched to the `boss` user):

{% highlight properties %}
# disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
{% endhighlight %}

After a system reboot the output of
`cat /proc/sys/net/ipv6/conf/all/disable_ipv6` should be `1`, meaning that IPV6
is actually disabled.

### Hadoop

#### Download and install Hadoop

Download
[hadoop-0.20.205.0.tar.gz](http://archive.apache.org/dist/hadoop/core/hadoop-0.20.205.0/hadoop-0.20.205.0.tar.gz),
unpack it and move the results in `/usr/local`, adding a symlink using the more
friendly name `hadoop` and changing ownership to the `hduser` user:

{% highlight console %}
boss@manhattan:~$ wget http://archive.apache.org/dist/hadoop/core/hadoop-0.20.205.0/hadoop-0.20.205.0.tar.gz
...
boss@manhattan:~$ sudo tar xzf hadoop-0.20.205.0.tar.gz
boss@manhattan:~$ sudo mv hadoop-0.20.205.0 /usr/local/hadoop-0.20.205.0
boss@manhattan:~$ cd /usr/local
boss@manhattan:/usr/local$ sudo ln -s hadoop-0.20.205.0 hadoop
boss@manhattan:/usr/local$ sudo chown -R hduser:hadoop hadoop
{% endhighlight %}

#### Setup the dedicated user environment

Switch to the `hduser` user and add the following lines at the end of the
`.bashrc` file:

{% highlight sh %}
# Set Hadoop-related environment variables
export HADOOP_PREFIX=/usr/local/hadoop

# Set JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/jdk1.7.0

# Some convenient aliases and functions for running Hadoop-related commands
unalias fs &> /dev/null
alias fs="hadoop fs"
unalias hls &> /dev/null
alias hls="fs -ls"

# Add Hadoop bin/ directory to PATH
export PATH=$PATH:$HADOOP_PREFIX/bin
{% endhighlight %}

get back to the administrator user, then open
`/usr/local/hadoop/conf/hadoop-env.sh`, uncomment the line setting `JAVA_HOME`
and set its value to the jdk directory:

{% highlight sh %}
export JAVA_HOME=/usr/lib/jvm/jdk1.7.0
{% endhighlight %}

#### Configure Hadoop
First of all, a directory for temporary data generated by Hadoop should be in
place, with proper access rights:

{% highlight console %}
boss@manhattan:~$ sudo mkdir -p /opt/hadoop/tmp
boss@manhattan:~$ sudo chown hduser:hadoop /opt/hadoop/tmp
boss@manhattan:~$ sudo chmod 750 /opt/hadoop/tmp
{% endhighlight %}

This directory should be specified as value for the `hadoop.tmp.dir` property
in file `/usr/local/hadoop/conf/core-site.xml`. Note that this file will
likely contain only an empty `configuration` tag, within which a `property` tag
should be nested:

{% highlight xml %}
<!-- Put site-specific property overrides in this file. -->
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/hadoop/tmp</value>
    <description>A base for other temporary directories</description>
  </property>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:54310</value>
    <description>The name of the default file system. A URI whose scheme and
    authority determine the FileSystem implementation. The URI's scheme
    determines the config property (fs.SCHEME.impl) naming the FileSystem
    implementation class. The URI's authority is used to determione the host,
    port, etc. for a file system.</description>
  </property>
</configuration>
{% endhighlight %}

The configuration process also requires to add a `mapred.job.tracker` property
in `/usr/local/hadoop/conf/mapred-site.xml`

{% highlight xml %}
<!-- Put site-specific property overrides in this file. -->
<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>localhost:54311</value>
    <description>The host and port that the MapReduce job tracker runs at. If
    "local", then jobs are run in-process as a single map and reduce tasks.
    </description>
  </property>
</configuration>
{% endhighlight %}

and a `dfs.replication` property in `/usr/local/hadoop/conf/hdfs-site.xml`

{% highlight xml %}
<!-- Put site-specific property overrides in this file. -->
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
    <description>Default block replication. The actual number of replications
    can be specified when the file is created. The default is used if
    replication is not specified in create time.</description>
  </property>
</configuration>
{% endhighlight %}

#### Formatting the distributed file system
The last step consists in formatting the file system, operation to be
executed as `hduser`:

{% highlight console %}
hduser@manhattan:~$ /usr/local/hadoop/bin/hadoop namenode -format
23/04/13 16:59:56 INFO namenode.NameNode: STARTUP_MSG:
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = manhattan/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 0.20.205.0
STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-0.20-security-205 -r 1179940; compiled by 'hortonfo' on Fri Oct 7 06:20:32 UTC 2011
************************************************************/
23/04/13 16:59:56 INFO util.GSet: VM type       = 64-bit
23/04/13 16:59:56 INFO util.GSet: 2% max memory = 19.33375 MB
23/04/13 16:59:56 INFO util.GSet: capacity      = 2^21 = 2097152 entries
23/04/13 16:59:57 INFO namenode.FSNamesystem: fsOwner=hduser
23/04/13 16:59:57 INFO namenode.FSNamesystem: supergroup=supergroup
23/04/13 16:59:57 INFO namenode.FSNamesystem: isPermissionEnabled=true
23/04/13 16:59:57 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
23/04/13 16:59:57 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
23/04/13 16:59:57 INFO common.Storage: Image file of size 112 saved in 0 seconds.
23/04/13 16:59:57 INFO common.Storage: Storage directory /opt/hadoop/tmp/dfs/name has been successfully formatted.
23/04/13 16:59:56 INFO namenode.NameNode: SHUTDOWN_MSG:
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at manhattan/127.0.1.1
************************************************************/
hduser@manhattan:~$
{% endhighlight %}

#### And... that's it!
Hadoop is now installed. The scripts `/usr/local/hadoop/bin/start-all.sh` and
`/usr/local/hadoop/bin/stop-all.sh` respectively start and stop all processes
related to Hadoop.
