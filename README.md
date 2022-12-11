 #### Logging in Linux systems, rely on a daemon to facilitate logging. Rsyslogd is the name of the reliable old service and it is open-source.

```
[root@ip-172-31-80-39 logrotate.d]# rsyslogd -v
rsyslogd  8.2102.0-105.el9 (aka 2021.02) compiled with:
	PLATFORM:				x86_64-redhat-linux-gnu
	PLATFORM (lsb_release -d):		
	FEATURE_REGEXP:				Yes
	GSSAPI Kerberos 5 support:		Yes
	FEATURE_DEBUG (debug build, slow code):	No
	32bit Atomic operations supported:	Yes
	64bit Atomic operations supported:	Yes
	memory allocator:			system default
	Runtime Instrumentation (slow code):	No
	uuid support:				Yes
	systemd support:			Yes
	Config file:				/etc/rsyslog.conf
	PID file:				/var/run/rsyslogd.pid
	Number of Bits in RainerScript integers: 64


[root@ip-172-31-80-39 logrotate.d]# systemctl status rsyslog
● rsyslog.service - System Logging Service
     Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-12-11 07:07:43 UTC; 27min ago
       Docs: man:rsyslogd(8)
             https://www.rsyslog.com/doc/
   Main PID: 628 (rsyslogd)
      Tasks: 3 (limit: 4384)
     Memory: 4.1M
        CPU: 128ms
     CGroup: /system.slice/rsyslog.service
             └─628 /usr/sbin/rsyslogd -n

Dec 11 07:07:43 localhost systemd[1]: Starting System Logging Service...
Dec 11 07:07:43 localhost systemd[1]: Started System Logging Service.
Dec 11 07:07:43 localhost rsyslogd[628]: [origin software="rsyslogd" swVersion="8.2102.0-105.el>
Dec 11 07:07:43 localhost rsyslogd[628]: imjournal: No statefile exists, /var/lib/rsyslog/imjou>
Dec 11 07:07:43 localhost rsyslogd[628]: imjournal: journal files changed, reloading...  [v8.
```


 #### Logrotate is a Linux utility whose core function is to rotate logs. If it is not installed as part of the default OS installation, it can be installed simply by running:

 ```
 yum install logrotate
 ```
 

 #### The first file to take note of with regard to the function of logrotate is logrotate.conf. This config file contains the directives for how log files are to be rotated by default. If there is no specific set of directives, the utility acts according to the directives in this file.

```
[root@ip-172-31-80-39 etc]# cat logrotate.conf 
# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may be also be configured here.
```



 #### The /etc/logrotate.d directory contains configuration files related to specific services like apache 
 
 ```
 [root@ip-172-31-80-39 logrotate.d]# cat httpd 
# Note that logs are not compressed unless "compress" is configured,
# which can be done either here or globally in /etc/logrotate.conf.
/var/log/httpd/*log {
    missingok
    notifempty
    sharedscripts
    delaycompress
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
```


```
notifempty - Directive stating that the file should not be rotated if it is empty.

Sharedscripts -  The sharedscripts means that the postrotate script will only be run once (after the old logs have been compressed), not once for each log which is rotated. 


missingok - If the log file is missing, go on to the next one without issuing an error message. 

delaycompress - Postpone compression of the previous log file to the next rotation cycle. This only has effect when used in combination with compress. It can be used when some program cannot be told to close its logfile and thus might continue writing to the previous log file for some time.

The lines between postrotate and endscript (both of which must appear on lines by themselves) are executed (using /bin/sh) after the log file is rotated.
```

 #### Excecuting a dry run of a specific service log configuation file 
 
```
root@ip-172-31-80-39 logrotate.d]# logrotate -d /etc/logrotate.d/httpd 
WARNING: logrotate in debug mode does nothing except printing debug messages!  Consider using verbose mode (-v) instead if this is not what you want.

reading config file /etc/logrotate.d/httpd
Reading state from file: /var/lib/logrotate/logrotate.status
error: error opening state file /var/lib/logrotate/logrotate.status: No such file or directory
Allocating hash table for state file, size 64 entries

Handling 1 logs

rotating pattern: /var/log/httpd/*log  1048576 bytes (no old logs will be kept)
empty log files are not rotated, old logs are removed
considering log /var/log/httpd/*log
  log /var/log/httpd/*log does not exist -- skipping
Creating new state
not running postrotate script, since no logs were rotated
```
