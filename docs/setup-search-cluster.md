Setup Search Head Clustering
============================

References:
- [Search Head Cluster Deployment Overview](https://docs.splunk.com/Documentation/Splunk/8.0.2/DistSearch/SHCdeploymentoverview)


Steps:
1. Setup Deployer
2. Setup Cluster Members
3. Setup Cluster Captain
4. Check Cluster Status
5. Connecting To Indexer Cluster


Setup Deployer
--------------
The deployer is used for distributing apps and updated configurations to the cluster members.
Within the reference architecture the deployer role is provided by the `splunk-mgt` instance.

Important:
- Remember the `pass4SymmKey` passphrase as you need to set it up on all the search heads.

Configure the deployer using the following `/opt/splunk/etc/system/local/server.conf` settings:
```
[shclustering]
pass4SymmKey = whatever
shcluster_label = shcluster1
```

After applying the settings you need to restart Splunk: 
`systemctl restart splunkd` or `/etc/init.d/splunk restart`


Setup Cluster Members
---------------------
This step should be performed on all search heads as refered to as `splunk-shN` in the reference architecture.

Configure each search head to be a cluster member using the following `/opt/splunk/etc/system/local/server.conf` settings:
```
[replication_port://34567]

[shclustering]
conf_deploy_fetch_url = https://splunk-mgt:8089
disabled = 0
mgmt_uri = https://splunk-sh1:8089
pass4SymmKey = whatever
replication_factor = 2
shcluster_label = shcluster1
```

Alternatively you can use the CLI to configure the search head cluster member:
`splunk init shcluster-config -auth admin:password -mgmt_uri https://splunk-shN:8089 -replication_port 34567 -replication_factor 2 -conf_deploy_fetch_url https://splunk-mgt:8089 -secret whatever -shcluster_label shcluster1`

After applying the settings you need to restart Splunk:
`systemctl restart splunkd` or `/etc/init.d/splunk restart`


Setup Cluster Captain
---------------------
Select *one* of the initialized instances to be the cluster captain. It does not matter which instance you select for this role, but in this example the `splunk-sh1` will be used.

`splunk bootstrap shcluster-captain -servers_list "https://splunk-sh1:8089,https://splunk-sh2:8089,https://splunk-sh3:8089,https://splunk-shN:8089" -auth admin:password`


Check Cluster Status
--------------------
Some useful commands to verify the health and/or configuration of the search head cluster:
- cluster status: `splunk show shcluster-status`
- cluster members: `splunk list shcluster-members`
- kv store status: `splunk show kvstore-status`


Connecting To Indexer Cluster
-----------------------------
The final step is to have all the search head cluster members connect to the index cluster so that we can actually start searching.
This step should be performed on all search heads as refered to as `splunk-shN` in the reference architecture.

Important:
- The password (pass4SymmKey or -secret) should correspond with the password used for setting up the indexer cluster.

Configure each search head cluster member to connect to the indexer cluster using the following `/opt/splunk/etc/system/local/server.conf` settings:
```
[clustering]
master_uri = https://splunk-mgt:8089
mode = searchhead
pass4SymmKey = whatever
```

Alternatively you can use the CLI to connect to the indexer cluster:
`splunk edit cluster-config -mode searchhead -master_uri https://splunk-mgt:8089 -secret whatever`

After applying the settings you need to restart Splunk:
`systemctl restart splunkd` or `/etc/init.d/splunk restart`
