This example includes components of a oozie workflow - scripts/code, sample data
and commands; Oozie actions covered: java mapreduce action; Oozie controls
covered: start, kill, end; The java program uses regex to parse the logs, and
also extracts the path of the mapper input directory path and includes in the
key emitted.
 
Note: The reducer can be specified as a combiner as well.
Usecase
-------
Parse Syslog generated log files to generate reports;

Includes:
---------
Sample data and structure:    01-SampleDataAndStructure
Data and script download:     02-DataAndScriptDownload
Data load commands:           03-HdfsLoadCommands
Java MR - Mapper code:        04A-MapperJavaCode
Java MR - Reducer code:       04B-ReducerJavaCode
Java MR - Driver code:        04C-DriverJavaCode
Command to test Java MR program: 04D-CommandTestJavaMRProg
Oozie job properties file:    05-OozieJobProperties
Oozie workflow file:          06-OozieWorkflowXML
Oozie commands                07-OozieJobExecutionCommands
Output -Report1               08-OutputOfJavaProgram


01a. Sample data
-----------------
May 3 11:52:54 cdh-dn03 init: tty (/dev/tty6) main process (1208) killed by TERM signal
May 3 11:53:31 cdh-dn03 kernel: registered taskstats version 1
May 3 11:53:31 cdh-dn03 kernel: sr0: scsi3-mmc drive: 32x/32x xa/form2 tray
May 3 11:53:31 cdh-dn03 kernel: piix4_smbus 0000:00:07.0: SMBus base address uninitialized - upgrade BIOS or use force_addr=0xaddr
May 3 11:53:31 cdh-dn03 kernel: nf_conntrack version 0.5.0 (7972 buckets, 31888 max)
May 3 11:53:57 cdh-dn03 kernel: hrtimer: interrupt took 11250457 ns
May 3 11:53:59 cdh-dn03 ntpd_initres[1705]: host name not found: 0.rhel.pool.ntp.org
01b. Structure
---------------
Month = May
Day = 3
Time = 11:52:54
Node = cdh-dn03
Process = init:
Log msg = tty (/dev/tty6) main process (1208) killed by TERM signal


-------------------
Directory structure
-------------------
oozieProject
	data
		kashit-syslog
			<<Node-Name>>
				<<Year>>
					<<Month>>
						messages
	
	workflowJavaMapReduceAction
		workflow.xml
		job.properties
	lib
		LogEventCount.jar


03-Hdfs load commands
----------------------
$ hadoop fs -mkdir oozieProject
$ hadoop fs -put oozieProject/* oozieProject/

04D Commands to test the java program in isolation
-----------------------------------------------
a) Command to run the program
$ hadoop jar oozieProject/workflowJavaMapReduceAction/lib/LogEventCount.jar kashit.oozie.samples.LogEventCount "oozieProject/data/*/*/*/*/*" "oozieProject/workflowJavaMapReduceAction/myCLIOutput"
b) Command to view results
$ hadoop fs -cat oozieProject/workflowJavaMapReduceAction/myCLIOutput/part* | sort
 
 
c) Results
2013-NetworkManager 7
22013-console-kit-daemon 7
2013-gnome-session 11
2013-init 166
2013-kernel 810
2013-login 2
2013-NetworkManager 7
2013-nm-dispatcher.action 4
2013-ntpd_initres 4133
2013-polkit-agent-helper-1 8
2013-pulseaudio 18
2013-spice-vdagent 15
2013-sshd 6
2013-sudo 8
2013-udevd 6



07. Oozie commands
-------------------
Note: Replace oozie server and port, with your cluster-specific.
1) Submit job:
$ oozie job -oozie http://cdh-dev01:11000/oozie -config oozieProject/workflowJavaMapReduceAction/job.properties -submit
job: 0000012-130712212133144-oozie-oozi-W
2) Run job:
$ oozie job -oozie http://cdh-dev01:11000/oozie -start 0000014-130712212133144-oozie-oozi-W
3) Check the status:
$ oozie job -oozie http://cdh-dev01:11000/oozie -info 0000014-130712212133144-oozie-oozi-W
4) Suspend workflow:
$ oozie job -oozie http://cdh-dev01:11000/oozie -suspend 0000014-130712212133144-oozie-oozi-W
5) Resume workflow:
$ oozie job -oozie http://cdh-dev01:11000/oozie -resume 0000014-130712212133144-oozie-oozi-W
6) Re-run workflow:
$ oozie job -oozie http://cdh-dev01:11000/oozie -config oozieProject/workflowJavaMapReduceAction/job.properties -rerun 0000014-130712212133144-oozie-oozi-W
7) Should you need to kill the job:
$ oozie job -oozie http://cdh-dev01:11000/oozie -kill 0000014-130712212133144-oozie-oozi-W
8) View server logs:
$ oozie job -oozie http://cdh-dev01:11000/oozie -logs 0000014-130712212133144-oozie-oozi-W
Logs are available at:
/var/log/oozie on the Oozie server.


08-Workflow application output
-------------------------------
 
$ hadoop fs -ls -R oozieProject/workflowJavaMapReduceAction/output/part* | awk '{print $8}'
oozieProject/workflowJavaMapReduceAction/output/part-r-00000
oozieProject/workflowJavaMapReduceAction/output/part-r-00001
 
$ hadoop fs -cat oozieProject/workflowJavaMapReduceAction/output/part*
2013-init 166
2013-polkit-agent-helper-1 8
2013-spice-vdagent 15
2013-sshd 6
2013-udevd 6
2013-NetworkManager 7
2013-console-kit-daemon 7
2013-gnome-session 11
2013-kernel 810
2013-login 2
2013-nm-dispatcher.action 4
2013-ntpd_initres 4133
2013-pulseaudio 18
2013-sudo 8