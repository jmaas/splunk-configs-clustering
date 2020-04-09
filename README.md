
This repository contains several basic configuration files required
by recent Splunk versions. Version used for testing is Splunk 8.0.2.1
The instructions and files included in this repository allow you to 
set-up a clustered environment relatively easy.

Reference Architecture
======================
This repository is based on an architecture I have running in my home lab.
The references to the hostnames should be replaced with the equivalents in your environment.

- 3x indexers (`splunk-idxN`)
- 3x search head (`splunk-shN`)
- 1x universal forwarder (`splunk-ufN`)
- 1x management server (`splunk-mgt`) with roles:
	- license master
	- cluster master
	- monitoring console
	- deployer

Manual Installation
===================
Repeat all steps for every Splunk instance type in your architecture except for the Universal
Forwarder instance (`splunk-ufN)`.

Splunk Enterprise
-----------------
- install Splunk Enterprise using the package (rpm, deb, tgz) that best fits your environment
- switch to splunk user `sudo su - splunk` (when using tgz create user/group manually first)
- accept license and setup the admin account `$SPLUNK_HOME/bin/splunk start --accept-license` 
- stop splunk `$SPLUNK_HOME/bin/splunk stop`

Systemd based systems
---------------------
- copy `systemd/disable-thp.service` over to `/etc/systemd/system/`
- copy `systemd/splunkd.service` over to `/etc/systemd/system/`
- make sure you don't `enable boot-start`, just to be sure `rm -f /etc/init.d/splunk`
- reload systemd unit files from disk `systemctl daemon-reload`
- enable the disable-thp service `systemctl enable disable-thp.service`
- enable the splunkd service `systemctl enable splunkd.service`
- start splunk `systemctl start splunkd.service`

Sysvinit based systems
----------------------
- copy `sysvinit/99-splunk.conf` over to `/etc/security/limits.d/`
- disable THP `echo sysvinit/rc.local >> /etc/rc.local`
- start splunk on boot `/opt/splunk/bin/splunk enable boot-start -user splunk`
- start splunk `/etc/init.d/splunk start`

Verification
------------
Verify that THP is disabled:
```
[splunk@splunk-mgt ~]$ cat /sys/kernel/mm/transparent_hugepage/defrag
always madvise [never]
[splunk@splunk-mgt ~]$ cat /sys/kernel/mm/transparent_hugepage/enabled
always madvise [never]
```

Verify that Splunk is not complaining about ulimits:
```
[splunk@splunk-mgt ~]$ grep limit /opt/splunk/var/log/splunk/splunkd.log | tail -n 12
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: virtual address space size: unlimited
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: data segment size: unlimited
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: resident memory size: unlimited
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: stack size: 8388608 bytes [hard maximum: unlimited]
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: core file size: 0 bytes [hard maximum: unlimited]
06-05-2018 19:44:01.122 +0200 WARN  ulimit - Core file generation disabled.
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: data file size: unlimited
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: open files: 64000 files
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: user processes: 16000 processes
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Limit: cpu time: unlimited
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Linux transparent hugepage support, enabled="never" defrag="never"
06-05-2018 19:44:01.122 +0200 INFO  ulimit - Linux vm.overcommit setting, value="0"
```

Configuration Apps
==================
Apps are used as configuration bundles for the different instance roles in your environment.

Master apps
-----------
These apps are installed on the `cluster master` in `/opt/splunk/etc/master-apps` and pushed to all indexers.

- `base_config_indexers`, configures: splunk-web, license master, inputs, volumes and indexes


Search-head Apps
----------------
The search head apps are installed on the `deployer` in `/opt/splunk/etc/shcluster/apps` and pushed to all search-heads.

- `xxx`:

Deployment apps
---------------
Deployment apps are installed on the `deployment server` in `/opt/splunk/etc/deployment-apps` and pushed to all forwarders.
 
- `base_config_forwarders`:

Notes
=====
Instructions for the most common tasks are provided in the `notes/` directory.

