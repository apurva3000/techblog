### Deploying Java SpringBoot code on Openshift

This post is divided into two parts:
1. Maven plugin : as a seasoned Java developer, a lot of people tend to use the maven assembly plugin for building their code. 

The plugin to use the boot plugin and the pom.xml should contain the following blocks

    <build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

Tip: As a novice, it is very easy to miss out on some of the dependencies required when using Spring Boot, so it is recommended to use the Spring Boot Initializer to start things off.

2. With the Maven plugin done, it's time to deploy stuff on Openshift, and for that you need to use the S2I (Source-to-image) utility which builds the image like magic.

However, the image which is recommended for building Java based images, might not actually work for some people (As one of my attempts, I tried *redhatopenjdk/redhat-openjdk18-openshift* image but it didn't work for me)
So, I found out the image to be used which works for building Java Spring Boot code is : `fabric8/s2i-java`

The command for triggering a build in Openshift is : `oc new-build --name=mongorest fabric8/s2i-java  --binary=true`
Notice, the option `binary` , it is used when you are building from a local directory instead of, for example - a Git repo.

### GIT chat: rebase
If you're looking to use git for real world applications and your team size is growing, chances are that you might have to start using rebase soon. This is used when the master of the repo has been updated since you started working on your branch, then you would have to merge the latest changes from the master to your own branch.

Without going too deep into all the details, I would mention my own experiences here. So, the first thing is that using a merge commit is ugly and ruins the master history, specially when the number of commits keep on increasing and it gets trickier to track everything. Also, rebase allows you see if you there any conflicts with your current branch to the latest master.

In order to do a rebase, make sure you have updated your master branch and then from your branch run a rebase like this:

`git rebase master` 

But the actual fun begins when you start doing complex things with an interactive rebase, in order to do a that, type the following:

`git rebase master -i`

Now you might see a list of commits like this:

pick 0x0x0x0 commit 1

pick 000x000 commit 2

pick 00000x0 commit 3

Now what you can do is move commits around by just editing the order in a text editor like vim, or use the options like "squash" , "fixup", "reword", "edit".

So, if you edit "pick" with "squash" in commit 2, it will merge the commit 2 to commit 1, as a single commit with the messages merged. Similarly, if you had used "fixup" in commit 2, it would have still merged the commit but erased the commit message of commit 2. 

These are used when you're working on complex issues and do not want to create messy looking history. Mail me in case you have doubts.

###  Fixing the NameNode Blocks Health alert in HDFS
If you're using Ambari, the red alert pops up as "NameNode Blocks Health" , and as per the description from Hortonworks "This service-level alert is triggered if the number of corrupt or missing blocks exceeds the configured critical threshold."
That being said, it means that the blocks in HDFS have gone corrupt. 

To check what files have been corrupted, execute the command (to be run with superuser privileges):
`sudo -u hdfs hadoop fsck -list-corruptfileblocks `

To fix the corrupted blocks, try to execute the following command:
`sudo -u hdfs hadoop fsck / -delete`

### GIT chat: reset
coming soon

### How to connect Jupyter ipython notebook to Spark running on standalone or Yarn
Coming soon.

### HortonWorks NFSGateway Configuration
NFSGateway allows you to browse your HDFS file system using a local file system using NFS. Not to forget, you could directly put files in HDFS by copying your files in the local file system.

If you happen to use NFSGateway for your Hortonworks cluster _(I am using HDP 2.3)_, you would often be puzzled about where to find the mount point location i.e. The filesystem where you can see all the files being kept in hdfs.
Although, Ambari will show you that NFSGateway is successfully running in the dashboard, it means it has only started the rpcbind and nfs services _(I run it on my master node which happens to be my namenode)_

So, you need to mount the hdfs to a location on your local filesystem manually. And here's how you can do it:

First create a directory named say for example, `/hdfs-local` (requires sudo rights)

Then Run the following command (Requires sudo rights)

`mount -t nfs -o vers=3,proto=tcp,nolock localhost:/ /hdfs-local`

NOTE: This assumes you are running the NFSGateway on the current node where you're trying to mount it (localhost used in the config)

### Repartition vs Coalesce
As you become familiar with Spark, you would come across a functionality called repartition. This essentially tries to reconfigure the number of partitions of your original RDD. You require them to increase or decrease the total number of partitions. For example, when the default number of partitions are quite high and thus during any operation - huge number of tasks are generated by Spark. Or, If the number of partitions are too few and in order to use the power of parallelism , you ensure that there are enough partitions of your RDD.

Repartitioning actually tries to move the data all over the network (shuffle mode on) to increase/decrease the number of partitions. Any shuffle operation is costly when it comes to large operations. Hence, a common suggestion is to try another functionality of Spark called 'Coalesce' - which tries to combine the number of partitions without actually performing the costly shuffle. 
The point to be noted is, this can be only used to decrease the number of partitions as compared to Repartition which can both increase or decrease it.

### Spark Shuffle tuning for optimal performance with Joins in Dataframes
It was often the case in my work that I used to perform joins with two dataframes (I used dataframes because they are much more optimized than the RDDs when it comes to shuffle operations), but since the join is essentially a cartesian join, the number of partitions would get multiplied and I would often land with huge number of tasks. 

They key here is to tune a parameter called `spark.sql.shuffle.partitions` which has a default value of 200. I tried lowering it and then successfully reduced the number of tasks which spark used to spawn.

### Performing Background tasks in Spring and Message Queues
Coming soon

### Using nginx to secure your web services better
Coming soon

### The Importance of Forward Slash in nginx
Often when using ngnix as a reverse proxy and providing the proxy pass as a URL to, for example, some service running on a webserver, there are some problems when the user tries to use the nginx url.

For example in this configuration

    location /resources {
        proxy_pass   http://tomcat-backend-server:8080/ResourceStatus/rest/status/;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

If I try to access the url by http://ngnix-proxy-server-address/resources , this would work fine, but if I or my broswer puts a forward slash http://ngnix-proxy-server-address/resources/, then this would fail as the mapping for tomcat backend server then becomes ResourceStatus/rest/status//, which is undefined. 
Hence, when providing the proxy_pass, remove the forward slash from the end of the url, to avoid any problems.

### Contact
Blog of @apurva3000 Check out my repos at [repo list](https://github.com/apurva3000)
