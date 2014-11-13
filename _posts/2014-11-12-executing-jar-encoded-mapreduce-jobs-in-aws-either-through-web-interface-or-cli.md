---
layout: post
title: "Executing JAR encoded MapReduce jobs in AWS either through Web interface or CLI"
description: ""
category: Big scale analytics
tags: [AWS, MapReduce, Hadoop, Java]
---
{% include JB/setup %}

This tutorial explains how to use AWS in order to run MapReduce jobs written in
Java end encoded into a JAR file, either using the Web-based console or through
a command line utility.

A basic knowledge of the Hadoop environment (release 2.4.0) and of the Hadoop
Java library is requested, as well as a fair proficiency in the use of a
terminal shell.

## Preliminary operations

### Sign up to AWS

AWS provides access to registered users, thus if you haven't already you need
to sign up at [http://aws.amazon.com](http://aws.amazon.com). The process is
relatively simple, but you can refer to several tutorials, such as [the one
provided by Sebastien Robaszkiewicz](http://snap.stanford.edu/class/cs341-2013/downloads/amazon-emr-tutorial.pdf).

> Be aware that AWS charges users for the provided services, and the sign up
> procedure requires to specify a valid credit card number. Although charges
> are low and per effective use, they can sum up to a considerable amount if
> some services are not esplicitly shut down. That said, there are several
> options in order to get free access: new AWS users can benefit of a 12-months
> no-charge period called [AWS Free Tier](http://aws.amazon.com/free/) for
> some services (unfortunately excluding Elastic MapReduce); moreover, you may
> benefit of promo codes (just keep on reading).


### Redeem promo codes (optional)

If you have one or more promo codes (for instance, each year AWS grants the
students of my [Big data analytics course](http://malchiodi.di.unimi.it/teaching/big-scale-analytics)
credit of US$ 100; if you are a teacher I strongly advice to refer to the
[AWS in Education Grants](http://aws.amazon.com/grants/) page), you can
redeem them from your AWS account page: log in to AWS and select the drop-down
menu showing your account name on the top of the page, then select "My account"
and "Credits", and finally type in your credit code and click the "Redeem"
button.


### Create a dedicated user

It is best not to use the so-called root account (the one automatically created
when signing up to AWS) for all activities. Specific tasks should be run by
dedicated users to be created and managed through AWS IAM (Identity and Access
Management). To do so, select "IAM" from the Services menu, then "Users" from
the left menu and click on the "Create New Users" button: type in `emr-develper`
in the first textbox and click the "Create" button at the bottom-right of the
page.

You will be prompted with the possibility to download the new user's security
credentials as a CSV file. Store them in a safe place (changing the default
name to `emr-developer-credentials.csv`) and click on the "Close" link at the
bottom of the page.

> The contents of this file should not be emailed or otherwise shared, for they
> will be used in order to secure your user's access to AWS. Moreover, once the
> file has been created it will not be possible to wiew the credentials through
> the IAM service anymore. Thus make sure to save this file in a safe and
> accessible place. In case you are using a shared computer, it is adivsable to
> `chmod 400` the file to ensure that other users cannot view its contents and
> the owner cannot accidentally delete or overwrite it.

You'll be prompted by a list of all users: click on the one just created and
click on the "Attach User Policy" button in order to give it permissions: in
the list under the "Select Policy Template" radio button, click the "Select"
button corresponding to "Amazon Elastic MapReduce Full Access" (you'll need to
scroll a bit to find it). Finally, click on the "Apply Policy" button.

## Some fundamental concepts in AWS

### The AWS Console

When signing in to AWS, the user is directed to a main page called the AWS
Console. This page shows all the available services, gathered into groups
focused, for instance, to computing, using databases and so on. The upper part
of the page, which is common to all the AWS services, there are several
drop-down menu buttons allowing the user to select a particular service, to
view and modify the user's account and to select specific AWS regions (see the
next section).

### AWS Regions

The AWS servers are located in different places worldwide, and grouped in
independent *regions*. Creation of a resource, say the computing nodes in a
cloud or some storage space in a distributed file system, should be attached to
a specific region, meaning that the resource will be hosted in one or more
servers in that specific region. Currently AWS provides the following regions:

- Asia Pacific, Tokyo (ap-northeast-1)
- Asia Pacific, Singapore (ap-southeast-1)
- Asia Pacific, Sydney (ap-southeast-2)
- EU, Frankfurt (eu-central-1)
- EU, Ireland (eu-west-1)
- South America, Sao Paulo (sa-east-1)
- US East, N. Virginia (us-east-1)
- US West, N. California (us-west-1)
- US West, Oregon (us-west-2)


Choosing a region close to your specific location results in a lower latency
time for data transfer, and can also meet specific legal regulations. Keep in
mind that transferring resources between different regions is charged on your
account, thus it is advisable to refer to the same region when using different
services (unless backing up stuff).

The upper right part of the AWS console always show the current region,
meanwhile allowing to select a new one. For the purposes of this tutorial,
we will keep the default region: `us-east-1`.

### The Simple Storage Service (S3)

The Simple Storage Service (S3, that is "S cubed", in the AWS terminology) gives
access to a distributed file system in order (among other things) to store data
to be used as input of our MapReduce jobs as well as to collect their outputs.

S3 gathers data in special containers called *buckets*, where a given bucket
(along with all its contents) is stored in servers belonging to a specific AWS
region. A bucket contains zero or more *objects*, which can be thought of as
the analogous of files and directories in a PC file system. Note that the
analogy holds only in an abstract way: at low level, the contents of an object
could be distributed over several machines; moreover, you can use objects that
appear as nested directories when viewed through the AWS console, but actually
all objects in a bucket are stored in a flat way, although each object is
identified by a *key name* which can be used to organize objects hyerarchically
through a specific namespace convention. Buckets and their contents are
univocally identified through a URI-like scheme using `s3` as protocol, also
accessible through a Web browser. Moreover, buckets can be associated to access
control lists in order to define which users can access them and how.

buckets are configured in order to be accessed from Web browsers using the
URL like

Connect to the S3 service in the AWS console and click on the "Create Bucket"
button. Select `us-east-1` region and type in a name for your bucket. Bucket
names should be unique across regions; moreover, there are several (both hard
and soft) [naming conventions](http://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html)
to be considered. Briefly put:

1. just use one or more words composed by characters, numbers, and hyphens,
   and use one single period to separate words;
2. use at least three and at most 63 characters.

For the purposes of this tutorial, let us pretend that you type in the name 
`emr-tutorial`, but in reality you will have to find up a bucket name not
already chosen which is easy for you to remember. Note that this bucket will
be accessible through the `s3://emr-tutorial` univocal name.

Once the "Create" button is clicked, the newly created bucket is listed along
with the ones previously created (thus, in case you signed up to follow this
tutorial you'll only see the `emr-tutorial` bucket). Clicking on the bucket
name has the effect of showing its contents and allowing standard file system
operations on them, as well as a direct data upload. In our case the
`emr-tutorial` bucket will be empty, and we need to add some folders to be used
later on: click the "Create Folder" button, type in the name `emr-logs` and
press return: the new folder will be shown among the contents of the `emr-logs`
bucket. Repeat this procedure in order to create the `emr-code` and the
`emr-input` folders, all directly contained in the `emr-tutorial`bucket.


### The Elastic Compute Cloud service (EC2)

The Elastic Compute Cloud service (EC2) provides computing facilities organized
in a reconfigurable cloud. You can think of such resources as a set of remote
virtual machines which you can configure and boot before logging in in order to
run computations. Each of these virtual machines can be set up in terms of
specific hardware and software, resulting in a specific AMI (Amazon Machine
Image) file. Several AMI are already available in order to meet the typical
requirements of AWS users.

In the context of MapReduce jobs, the cloud will contain nodes of a Hadoop
cluster according to a predefined AMI, and AWS will take care of booting all
nodes and executing the job.


### Access keys

Access to the AWS services is secured through two security credentials known as
*access key id* and *secret access key*. You can broadly think them as a login
name and a password. They are generated through the IAM service when creating a
user, and indeed your `emr-developer-credentials.csv` file exactly contains
such information for the `emr-developer` user.


## Writing the software and setting up input

In order to be able to execute a MapReduce job it is necessary to write and
compile the software to be run, and to gather up the data to be fed in input to
the job.

### Writing and compiling the software

The hello-world program in a MapReduce environment corresponds to a job
counting the frequencies of words in a set of textual documents. Such program
is composed by three Java classes, whose source is shown hereafter with
reference to the 2.4.0 release of Hadoop:

- the mapper class

{% highlight java %}
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	
    private final IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        StringTokenizer itr = new StringTokenizer(line.toLowerCase(), " .,:;!?{}[]()|_");
        while(itr.hasMoreTokens()) {
            word.set(itr.nextToken());
            context.write(word, one);
        }
    }
}
{% endhighlight %}

- the reducer class

{% highlight java %}
import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
	
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for(IntWritable v : values)
            sum += v.get();
        result.set(sum);

        context.write(key, result);
    }
}
{% endhighlight %}

- the driver class

{% highlight java %}
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class WordCount extends Configured implements Tool {

    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new Configuration(), new WordCount(), args);
        System.exit(res);
	}

    public int run(String args[]) {
        try {
            Configuration conf = new Configuration();

            Job job = Job.getInstance(conf);
            job.setJarByClass(WordCount.class);

            // specify a mapper
            job.setMapperClass(WordCountMapper.class);

            // specify a reducer
            job.setReducerClass(WordCountReducer.class);

            // specify output types
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(IntWritable.class);

            // specify input and output DIRECTORIES
            FileInputFormat.addInputPath(job, new Path(args[0]));
            job.setInputFormatClass(TextInputFormat.class);

            FileOutputFormat.setOutputPath(job, new Path(args[1]));
            job.setOutputFormatClass(TextOutputFormat.class);

            return(job.waitForCompletion(true) ? 0 : 1);
        } catch (InterruptedException|ClassNotFoundException|IOException e) {
            System.err.println("Error during mapreduce job.");
            e.printStackTrace();
            return 2;
        }
    }
}
{% endhighlight %}

Once these classes have been compiled including the Hadoop JAR library, the
produced bytecode should be exported into a JAR file whose main method is that
of the `WordCount` class. Both tasks are easily accomplished using a IDE such
as Eclipse. Save the JAR file on your local computer giving it the name
`wordcount.jar`. This file should be uploaded to S3 in order to be executed in
a cloud: log in to the AWS console and then connect to the S3 service, which
should list the `emr-tutorial`bucket. Click on the bucket name, then on the
`emr-code` folder, and finally on the "Upload" button: in the window which will
appear, click on "Add Files" and navigate to the `wordcount.jar` file. Finally,
click on the "Start Upload" button and wait until the upload completes.

### Setting up input

As input to our word-counting job we will use the text of two books: [Joyce's
Ulysses](https://www.gutenberg.org/ebooks/4300.txt.utf-8) and the first volume
of [Da Vinci's notebooks](https://www.gutenberg.org/ebooks/4998.txt.utf-8).
Download the two texts and subsequently upload them in the `emr-input` folder
in S3, following the same procedure of previous section. Note that the specific
names you give to the textual files is ininfluent to the job execution.

## Running a job: the AWS Console way

In order to run a job through the AWS Console you'll need to create a cluster,
submit the job, monitor the cluster until processing ends, retrieve results
and dispose the cluster.

### Creating a cluster and submitting a job

All things are in place in order to submit our job for execution: access the
Elastic MapReduce service (EMR) from the AWS console and click  on the "Create
cluster" button. There are several options available to tune how the
job will be executed (and—indirectly—how much you will be charged), which are
briefly revised hereafter.

#### Cluster Configuration

When submitting more jobs simultaneously, use the "Cluster name" textbox to
assign name to clusters, so as to be quickly able to distinguish them. For the
purposes of this tutorial, the default name can be kept.

Let the "Termination protection" selected (this will require an additional
user confirmation to avoid accidental shutdown of the cluster) and keep the
"Logging" and "Debugging" facilities. In the "Log folder" textbox type in
`s3://emr-tutorial/emr-logs`, or select the folder interactively through the
icon on the right of the textbox.

#### Tags

This section allows to tag clusters, for instance to quickly refer to those
related to a particular project. Leave everything unchanged.

#### Software Configuration

Here you can detail which software will be installed on your cluster nodes.
Keep the Amazon Hadoop distribution and the 3.3.0 AMI version. Delete the
three rows under "Applications to be installed" by clicking on the cross at
their right: as we will not be using either Hive, Pig, or Hue, installing them
would only result in longer time to wait (and higher charges).

#### File System Configuration

This section focuses on specific configurations on the cluster distributed file
system. Leave it as it is.

#### Hardware Configuration

Here you can shape your cluster, deciding about how many nodes it will contain
as well as on some hardware configuration of the various nodes. For this
tutorial, just use the default values, but keep in mind that in real life
applications an accurate tuning of the options in this section will
considerably affect the performances of your cluster.

#### Security and Access

Also in this case, use the default values, although in real life you will
likely decide to give specific IAM users permissions to use the cluster.
Moreover, in this section you can set up SSH access to nodes using a key pair
to be generated under the "Key pair section"

#### Bootstrap Actions

Here you can trigger the execution of specific script before Hadoop starts.
We will not use this facility.

#### Steps

This is the section where you specify how the cluster will execute your job.
From the "Add step" combo, select "Custom JAR" and subsequently "Configure and
add". In the dialog which is shown specify a meaningful name for the job or
leave the default value, and fill in the remaining field as follows:

- JAR location: either type in `s3://emr-tutorial/emr-code/wordcount.jar` or
  navigate through it interactively using the icon on the right;
- Arguments: type in `s3://emr-tutorial/emr-input s3://emr-tutorial/emr-output`.

Finally, click on the "Add" and the "Create cluster" buttons: your cluster will
be created and booted in order to run the MapReduce job described in the
supplied JAR file.

### Monitor the cluster

Once created, the cluster will automatically boot up, install additional
software and execute your job. In this process, the cluster status will evolve
from starting, to running and finally to waiting. Keep in mind that this will
take a fair amount of time, typically some minutes. During this time span, the
cluster status can be monitored from the EC2 service page.

### Retrieve results

Connect to the S3 service, opening a new browser tab or window, navigate from
the `emr-tutorial` bucket in the `emr-output` folder and view its contents: the
presence of a `_SUCCESS` file denotes that the MapReduce job terminated
correctly, and you'll find one ore more `part-r-nnnnn` files, `nnnnn` starting
from `00000` and increasing by one unit. The latter files contain the job
output, and they can be viewed locally after they have been downloaded through
either double clicking of the file name, selecting it and clicking on the
"Actions/Download" menu button in the upper part of the page, or finally
right-clicking on the file name and selecting the "Download" menu item. Once
the download completes, the files can be inspected locally.

If the output folder does not contain a `_SUCCESS` file, something went wrong,
thus some inspection of the logs created in the `emr-logs` folders should help
in bug fixing. A good starting point are the `stdout` and `stderr` files
containing all output sent to standard output and standard error. Finding them
requires to figure out the cluster ID (which is specified in the EMR page),
navigate to the corresponding directory within `emr-logs` and subsequently in
the `step` directory. Here you will find several directories, each related to
one of the steps executed in the cluster lifecycle. Look in the EMR page in
order to find out which ID corresponds to the "Custom JAR Step": the
corresponding directory will contain `stdout` and `stderr`.

### Dispose the cluster and delete unused files

Get back to the EMR service and click on the "Terminate" button. If you closed
the EMR window or tab when browsing the S3 buckets in order to view the job
results, just connect to the EMR service in the AWS console: you'll be prompted
with a list of recent clusters, the first being the one we just launched and
which is characterized by a "Waiting" status. Click on its name and then on the
"Terminate" button: a dialog will warn about the fact that the termination
procetction feature is enabled. To disable it, click on "Change", then on the
"Off" radio button, and finally on the green check mark. Now it will be
possible to click on the "Terminate" button in order to shut down the cluster.
Its status will turn into "Terminating", and after some minutes to "Terminated".

> Note that it is extremely important to shut down clusters when the job
> processing completes, in order to avoid additional charges on your account.

Analogously, remember to clean up unused stuff in your S3 bucket, such as logs
and, when processing ends the JAR and the input and output files (unless you
want to keep them in S3).


## EMR reloaded: the "command line" way

If you are the "avoid the GUI" type (I am!), you can use a tool conceived to access
AWS through the command line: the AWS Command Line Interface (AWS CLI for
short).

### Install the AWS CLI

The AWS command line interface can be installed in [several
ways](http://docs.aws.amazon.com/cli/latest/userguide/installing.html),
although on a Unix-like system some package manager is advisable: for instance

{% highlight console %}
$ brew install awscli
{% endhighlight %}

will install the CLI through homebrew on Mac OS X, although an equivalent
procedure can exploit `pip` in all Unix-like systems. Regardless of the chosen
installation procedure, if everithing goes smooth

{% highlight console %}
$ aws help
{% endhighlight %}

should display a man-like help page.


#### Configure the AWS CLI

To configure the AWS CLI, simply run

{% highlight console %}
$ aws configure
{% endhighlight %}

and insert the following data

- access key id: the second value in the `emr-developer-credentials.csv` file
  corresponding to the user created at the beginning of the tutorial;
- secret access key: the third value in the `emr-developer-credentials.csv`
  file;
- default region name: the name of thw AWS region you will use; type in
  `us-east-1`
- output format: type in `json`.

The latter choice sets how the AWS CLI will format the information received
from our cluster. Instead of a pure textual format, the use of
[JSON](http://www.json.org) will allow a easier parsing of the (typically
verbose) structures obtained as output. When interacting programmatically with
AWS the parsing will be typically done using specific libraries, but for the
purposes of this tutorial I suggest to `brew install` (or equivalent procedure)
the `jq` processor.

#### Set up command completion (optional)

If you use the shell command completion (and you should), you'll be happy to
add this feature to subcommands of the AWS CLI:

{% highlight console %}
$ export AWSBIN=$(dirname $(greadlink -f $(which aws)))
$ complete -C $AWSBIN/aws_completer aws
{% endhighlight %}

Note the use of `greadlink` in order to resolve the symbolic link to the AWS
binary: this is only necessary in Mac OS, as other Unix-like systems directly
allow to invoke `readlink`. Also note that `greadlink` is available only when
the GNU coreutils have been installed: if they weren't you can `brew install`
them. In all cases, `complete` can also be invoked directly, manually resolving
the absolute pathname of the AWS binary directory.

### Submitting a MapReduce job through the AWS CLI

If you are following this tutorial, input data and JAR are already in place.
But let's pretend they aren't, just to show how that the whole process can be
mastered through the AWS CLI. The first step is that of creating our bucket
using the `mb` command:

{% highlight console %}
$ aws s3 mb s3://emr-tutorial
{% endhighlight %}

The next operation concerns copying into S3 the job JAR and the input files:

{% highlight console %}
$ aws s3 cp ulysses.txt s3://emr-tutorial/emr-input
$ aws s3 cp notebooks.txt s3://emr-tutorial/emr-input
$ aws s3 cp wordcount.jar s3://emr-tutorial/emr-code
{% endhighlight %}

Remember that, although bucket contents in the AWS console are shown in a
hiearchical way, there are no directories: the console suitably *interpretes*
slashes in file names in order to visualize them hierarchically. This is why
we did not explicitly create directories (and, in fact, we couldn't have
because there is no AWS CLI command to do that).

Creating a cluster and submitting a JAR-encoded MapReduce job requires the
execution of the `create-cluster` command. As the number of parameters to be
specified is considerable, this command is typically invoked specifying several
options:

{% highlight console %}
$ aws emr create-cluster --ami-version 3.3.0 \
--instance-groups InstanceGroupType=MASTER, InstanceCount=1, InstanceType=m3.xlarge \
                  InstanceGroupType=CORE, InstanceCount=2, InstanceType=m3.xlarge \
--steps Type=CUSTOM_JAR, \
        Name="Custom JAR Step", \
        ActionOnFailure=CONTINUE, \
        Jar=s3://emr-tutorial/emr-code/wordcount.jar, \
        Args=["s3://emr-tutorial/emr-input", "s3://emr-tutorial/emr-output"] \
--no-auto-terminate \
--log-uri s3://emr-tutorial/emr-log \
--enable-debugging
{
    "ClusterId": "j-3JS43WVPKL4WC"
}
{% endhighlight %}

The obtained output is a JSON representation containing an ID to be used in
order to query AWS about the created cluster.

### Monitoring the cluster

The `describe-cluster` can be used in order to monitor the job execution,
specifying the cluster ID returned by `cluster-create`:

{% highlight console %}
$ aws emr describe-cluster --cluster-id "j-3JS43WVPKL4WC"
{
    "Cluster": {
        "Status": {
            "Timeline": {
                "ReadyDateTime": 1415823056.83, 
                "CreationDateTime": 1415822731.177
            }, 
            "State": "RUNNING", 
            "StateChangeReason": {
                "Message": "Running step"
            }
        }, 
        "Ec2InstanceAttributes": {
            "Ec2AvailabilityZone": "us-east-1d"
        }, 
        "Name": "Development Cluster", 
        "Tags": [], 
        "TerminationProtected": false, 
        "RunningAmiVersion": "3.3.0", 
        "Id": "j-3JS43WVPKL4WC", 
        "Applications": [
            {
                "Version": "2.4.0", 
                "Name": "hadoop"
            }
        ], 
        "MasterPublicDnsName": "ec2-54-161-146-73.compute-1.amazonaws.com", 
        "InstanceGroups": [
            {
                "RequestedInstanceCount": 1, 
                "Status": {
                    "Timeline": {
                        "ReadyDateTime": 1415822996.85, 
                        "CreationDateTime": 1415822731.178
                    }, 
                    "State": "RUNNING", 
                    "StateChangeReason": {
                        "Message": ""
                    }
                }, 
                "Name": "MASTER", 
                "InstanceGroupType": "MASTER", 
                "Id": "ig-3TT82IF5M4XRB", 
                "InstanceType": "m3.xlarge", 
                "Market": "ON_DEMAND", 
                "RunningInstanceCount": 1
            }, 
            {
                "RequestedInstanceCount": 2, 
                "Status": {
                    "Timeline": {
                        "ReadyDateTime": 1415823056.9, 
                        "CreationDateTime": 1415822731.178
                    }, 
                    "State": "RUNNING", 
                    "StateChangeReason": {
                        "Message": ""
                    }
                }, 
                "Name": "CORE", 
                "InstanceGroupType": "CORE", 
                "Id": "ig-1HWOUDNR20CHV", 
                "InstanceType": "m3.xlarge", 
                "Market": "ON_DEMAND", 
                "RunningInstanceCount": 2
            }
        ], 
        "VisibleToAllUsers": true, 
        "BootstrapActions": [], 
        "LogUri": "s3n://dmalchiodi/emr-log/", 
        "AutoTerminate": false, 
        "RequestedAmiVersion": "3.3.0"
    }
}
{% endhighlight %}

The command `describe-cluster` has a quite verbose output; being interested
only in querying the cluster status, contained in the `State` field of
the `Status` field nested within `Cluster`, we can use the `jq` utility
previously installed in order to filter out unwanted information:

{% highlight console %}
$ aws emr describe-cluster --cluster-id "j-3JS43WVPKL4WC" | jq '.Cluster.Status.State'
"RUNNING"
{% endhighlight %}

When the cluster status turns into "WAITING" you can explore the `emr-output`
folder in order to check whether the job did succeed:

{% highlight console %}
$ aws s3 ls s3://emr-tutorial/emr-output --recursive
2014-11-12 21:12:37          0 emr-output/_SUCCESS
2014-11-12 21:12:29      63145 emr-output/part-r-00000
2014-11-12 21:12:29      63902 emr-output/part-r-00001
2014-11-12 21:12:30      63228 emr-output/part-r-00002
2014-11-12 21:12:30      63653 emr-output/part-r-00003
2014-11-12 21:12:31      62845 emr-output/part-r-00004
2014-11-12 21:12:32      64917 emr-output/part-r-00005
2014-11-12 21:12:36      65097 emr-output/part-r-00006
{% endhighlight %}

If you don't find a `_SUCCESS` file, the computation did not complete
successfully, thus it is advisable to inspect the files in `emr-logs`.
Otherwise the `part-r-nnnnn` files can be downloaded for inspection.

{% highlight console %}
$ aws s3 cp s3://emr-tutorial/emr-output/part-r-00000 .
download: s3://emr-tutorial/emr-output/part-r-00000 to ./part-r-00000
$ cat part-r-00000 
"alpine-glow"   1
"as-is" 1
"bathers        1
"doctrinal      1
...
{% endhighlight %}

As a last step, the cluster has to be shut down through the
`terminate-clusters` command:

{% highlight console %}
$ aws emr terminate-clusters --cluster-ids "j-3JS43WVPKL4WC"
{% endhighlight %}

This step requires at least some minutes in order to complete, and also in
this case the `describe-cluster` command can be used in order to monitor how
the operation evolves:

{% highlight console %}
$ aws emr describe-cluster --cluster-id "j-3JS43WVPKL4WC" | jq '.Cluster.Status.State'
"TERMINATING"
{% endhighlight %}

When the status changes from `"TERMINATING"` to `"TERMINATED"` the cluster can
be considered safely shut down. As in previous section about AWS Console, at
this time you also should verify whether or not it is better to delete some of
the produced S3 content. 
