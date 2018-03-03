---
ID: 152
post_title: Running Cassandra In Netbeans
author: sathyak_1989
post_excerpt: ""
layout: post
permalink: >
  http://sathiyaprabhu.com/running-cassandra-netbeans/
published: true
post_date: 2014-10-06 07:16:21
---
Hi Forks, Today in this post am gonna tell how to open Cassandra source in Netbeans..

<img class="aligncenter wp-image-158 size-medium" src="http://sathiyaprabhu.com/look-inside/uploads/2014/10/Screen-Shot-2014-10-11-at-12.50.53-AM-300x138.png" alt="Running Cassandra in Netbeans" width="300" height="138" />

<span style="font-family: georgia, palatino;">It's been so long, since i started working on Cassandra till date whenever i tend to meet some students who newly started working on Cassandra, the common question which i came across is "How you managed to run the Cassandra source from Netbeans..?". Even i found these question is circulating online even in the Cassandra mailing list and nobody answered it.. So i took an opportunity to address it today in this post.. I hope this post would be helpful for the people who are annoyed seeing the Red Error icon trying long time to address the issue or definitely would be a time-cutter for the new people to get rid of the Red icon and start exploring/ running the Project from Netbeans..</span>

<span style="font-family: georgia, palatino;">Cassandra Wiki, gives the detailed information about opening the Cassandra source from Eclipse.. If we have eclipse project, we can easily import the Eclipse project from Netbeans..</span>

<span style="font-family: georgia, palatino;">Actually, the steps are very simple, takes less than a minute.. Please follow the simple steps below..</span>
<ul>
	<li><span style="font-family: georgia, palatino;">Download the latest Apache Cassandra source from the official website (http://cassandra.apache.org/download).</span></li>
	<li><span style="font-family: georgia, palatino;">After extracting the .tar file, navigate inside the extracted folder doing 'cd apache-cassandra-VERSION-src' (The latest version at this time is 2.1.0. So in my case it's apache-cassandra-2.1.0-src. To remain generic, am gonna say apache-cassandra-VERSION-src).</span></li>
	<li><span style="font-family: georgia, palatino;">Build the source project using ant 'ant build'.</span></li>
	<li><span style="font-family: georgia, palatino;">Generate the eclipse files 'ant generate-eclipse-files'.</span></li>
	<li><span style="font-family: georgia, palatino;">Open the Netbeans, hit File -&gt; Import Project -&gt; Eclipse Project ...</span></li>
	<li><span style="font-family: georgia, palatino;">When the Import Eclipse Project window pops up, choose 'Import Project ignoring Project Dependencies'</span></li>
	<li><span style="font-family: georgia, palatino;">Under Project to import, browse to the folder where you generated the eclipse files (apache-cassandra-VERSION-src).</span></li>
	<li><span style="font-family: georgia, palatino;">Under Destination Folder, you can either choose the same project folder where you generated the eclipse files (apache-cassandra-VERSION-src) or you can create a new folder where you keep all your Netbeans Projects  and choose that location.</span></li>
	<li><span style="font-family: georgia, palatino;">Wait for few minutes to let Netbeans to finish the Background scanning.</span></li>
	<li><span style="font-family: georgia, palatino;">Now Probably, Netbeans shows the annoying Red Error icon.. Don't worry we gonna fix it shortly by adding the needed jar files.</span></li>
	<li><span style="font-family: georgia, palatino;">Before adding the needed jar files, right click on the project folder and open the project properties.</span></li>
	<li><span style="font-family: georgia, palatino;">Under the 'Sources' category, choose 'Source/Binary Format' as JDK 7 and hit OK.</span></li>
	<li><span style="font-family: georgia, palatino;">Now go on to add the needed jar files. Right click on the 'Libraries' folder and choose 'Add JAR/Folder'.</span></li>
	<li><span style="font-family: georgia, palatino;">When the 'Add JAR/Folder' wizard pops up carefully choose the jar files from the following two folders.</span>
<ul>
	<li><span style="font-family: georgia, palatino;">All the jar files from the apache-cassandra-VERSION-src/lib. And Yes, you need only the jar files you can ignore the .zip files and licenses folder.</span></li>
	<li><span style="font-family: georgia, palatino;">All the jar files from the apache-cassandra-VERSION-src/build/lib/jars.</span></li>
</ul>
</li>
	<li><span style="font-family: georgia, palatino;">Now probably the annoying Red Error icon would have been disappeared.. Cool isn't it..? Don't hurry just one more step to finish the process.</span></li>
	<li><span style="font-family: georgia, palatino;">Currently, if you run the project, Netbeans choose the cassandra.yaml file from the test folders as the project configuration. Or the probable error would be "Fatal configuration error. Cannot locate cassandra.yaml." So,</span></li>
	<li><span style="font-family: georgia, palatino;">Right click on the 'Libraries' folder choose 'Add JAR.Folder' option and choose the conf folder from apache-cassandra-VERSION-src/conf.</span></li>
	<li><span style="font-family: georgia, palatino;">Configure the cassandra.yaml file mentioning the correct location for 'data_file_directories:', 'commitlog_directory:',  'saved_caches_directory:'and the '- seeds: ' if you are deploying Cassandra under multi-node cluster. Remember, by default, Cassandra tries to writes files under '/var/lib/cassandra/data' which need sudo access. So it's better to choose the location (may be the same project folder) which no need sudo access.</span></li>
	<li><span style="font-family: georgia, palatino;">And that's all, save the file..</span></li>
</ul>
<span style="font-family: georgia, palatino;">Enjoy, Play with the source and have fun...!</span>