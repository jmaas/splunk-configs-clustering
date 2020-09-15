# Setup Universal Forwarder

References:
- [Forwarder Manual](https://docs.splunk.com/Documentation/Forwarder/8.0.2/Forwarder/Configuretheuniversalforwarder)
- [Remote installation of the forwarder](http://docs.splunk.com/Documentation/Forwarder/latest/Forwarder/Installanixuniversalforwarderremotelywithastaticconfiguration)


Steps:
1. Software Installation
2. Initial Configuration
3. Setup Central Management
4. Verify Status


## Installation
This step documents the process of installing the Splunk Universal Forwarder software
onto the designated instance within the reference architecture (`splunk-ufN`).

I prefer to use proper packages as they provide some additional benefits from an 
operational perspective, in this example I use the RPM package.

```
[root@syslog tmp]# rpm -i ./splunkforwarder-8.0.3-a6754d8441bf-linux-2.6-x86_64.rpm
[root@syslog tmp]#
```


## Initial Configuration
For security purposes the agent should be run with least privilege, so never as root!
Conveniently the  package install process automatically created a group and user named `splunk`. 

To setup the agent to automatically start at boot and configure the administrator account 
execute the following commands as user `root`:

```
[root@syslog tmp]# cd /opt/splunkforwarder/bin
[root@syslog bin]# ./splunk enable boot-start -user splunk --accept-license

This appears to be your first time running this version of Splunk.

Splunk software must create an administrator account during startup. Otherwise, you cannot log in.
Create credentials for the administrator account.
Characters do not appear on the screen when you type in credentials.

Please enter an administrator username: admin
Password must contain at least:
   * 8 total printable ASCII character(s).
Please enter a new password: 
Please confirm new password: 
Init script installed at /etc/init.d/splunk.
Init script is configured to run at boot.
[root@syslog bin]# 
```

To start and verify the Splunk Universal Forwarder agent:
```
[root@syslog bin]# /etc/init.d/splunk start
Starting splunk (via systemctl):                           [  OK  ]
[root@syslog bin]# ps wwuax|grep splunk
splunk    4242 14.0 10.4 247112 87888 ?        Sl   10:42   0:00 splunkd -p 8089 start
splunk    4249  0.0  2.0  89836 17548 ?        Ss   10:42   0:00 [splunkd pid=4242] splunkd -p 8089 start [process-runner]
root      4303  0.0  0.1  12108  1124 pts/0    R+   10:43   0:00 grep --color=auto splunk
[root@syslog bin]# 
```


## Setup Central Management
The goal here is to have the least amount of configuration on the Universal Forwarder
endpoints and to have maximum control by leveraging the Remote Management capabilities. 

So all we need to do is pair this Universal Forwarder with the Deployment Server:
```
[root@syslog bin]# sudo su - splunk
[splunk@syslog ~]$ cd /opt/splunkforwarder/bin
[splunk@syslog bin]$ ./splunk set deploy-poll splunk-mgt:8089
Splunk username: admin
Password: 
Configuration updated.
[splunk@syslog bin]$ 
```


## Verify Status
- Check the Forwarder Management section in the webinterface of the `splunk-mgt` instance, it should list the Universal Forwarder we've added.
