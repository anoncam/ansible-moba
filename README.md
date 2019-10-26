# MicroFocus Operations Bridge Analytics

## AWS Configurations


```yaml
vpc_cidr_block: 10.2.13.0/28
```
Operating System | Version |   arch   |
-----------------|---------|----------|
RHEL             |   7.x   |  64-bit  |

**Baseline Dependencies**

* `m4`
 If the `/usr/bin/m4` executable is not present, install the m4 (m4.x86_64) package on your target server before proceeding with the OBA installation.
* Vertica Database
* `OpenJDK 1.8.0_121`
* JBOSS
* OBA Application Server
* OBA Control Server

**Infrastructure Requirements**

* Vertica Database Server
* OBA Control application server
* OBA Application server

_network information is contained further in documentation_

`The following will be updated 07/18/2019:` 

| Setup | RAM | Disk | AWS Instance Type|
|-------|-----|------|------------------|

**Instance Sizing**



**Environment**
- variables
```bash
OPSA_HOME=/opt/HP/opsa
JAVA_HOME=/opt/HP/opsa/jdk
```
- System

```bash
# This is the system file structure needed for OBA. Ensure no system variables override. 

#JBOSS Home
/opt/HP/opsa/jboss

#JDK
/opt/HP/opsa/jdk

# scripts
/opt/HP/opsa/scripts

# conf
/opt/HP/opsa/conf

# data
/opt/HP/opsa/data

# log
/opt/HP/opsa/log

# lib
/opt/HP/opsa/lib

# bin
/opt/HP/opsa/bin

# Vertica Database installation
/opt/vertica

# Operations Agent
/opt/OV

```
# Pre-Installation Tasks

* Meet the hardware and software requirements
* Overview of OBA components
* Map Ports
* Collection Source Types
* Overview of installation workflow
* Prepare to install OBA in a non-default location

**Meet the hardware and software requirements**

**Deployment**
While 3.01 is delivered as a patch, it should be installed on a fresh system prior to integration with Ansible Tower.

* Ensure that the required dialog packages are installed on the hosts you plan to use for the Vertica Database you plan to use for Operations Bridge Analytics:
* You should see output from the following command: rpm -qa | grep dialog
* If you run this command on a server running RH6.7, you should see a result similar to the following: dialog-1.1-9.20080819.1.el6.x86_6
* If you run this command on a server running RH7.x, you should see a result similar to the following: dialog-1.1-9.20080819.1.el6.x86_6
* If you do not see any result from the previous command, you do not have a dialog package installed. You must install it before continuing. 
* Depending on your configuration you might be able to install it using a command similar to the following:
`yum install dialog`
* For the instructions in this manual, use the information in Available Downloads for Operations Bridge Analytics to obtain the installation packages. 
* Download `OPSA_00010.zip` from Software Support Online (SSO). Extract the archive.

**Download Links**

| Download File Name | Purpose |
|--------------------|---------|
| HPE_OpsA_3.01_Analytics_Installation.zip | Operations Bridge AnalyticsOperations Bridge Analytics Linux Installations |
|------------------------------------------|----------------------------------------------------------------------------|
| HPE_OpsA_3.01_Vertica_Installation.zip   | Operations Bridge Analytics Vertica Installations                          |



[]()
[]()
[]()

**Overview of OBA components**

* **OBA Application Server**
    * biz logic and presentation functions of OBA
    * deployed as a server installation
    * OBA can have one or more OBA Application Server hosts, depending on your requirements for High Availability. See the section for details.
    * The application server is JBOSS based.
* **OBA Collector Host**
    * Connects to the different data Source Types and aggregates the data collected from them.
    * Posts to OBA database.
    * deployed as a server installation
    * OBA can have one or more OBA Collector hosts, depending on the expected volume of data from the data Source Types to which the system is connected.
    * Your installed collectors form a Kafka cluster. If your cluster consists of more than one collector, the replication factor is set to two by default. This means that the cluster can handle the loss of one collector. However, the cluster cannot operate anymore if, for example, more than one of your collectors are down due to maintenance. If, in that case, the collectors cannot be recovered, you must take disaster recovery actions.
* **OBA Database**
    * A Vertica database is used to support the big data analysis requirements of OBA.
    * An existing Vertica database installation can be used. The OBA database (`opsadb`) needs to be created on it.
    * A dedicated Vertica database can also be installed as part of OBA. In this case the OBA database (`opsadb`) will be created during the process.


**Map Ports (External Sources to OBA)**

| Port | Open From | Open To | Comments |
|------|-----------|---------|----------|
| 137  | OBA Collector host | NNM iSPI Performance for metrics and NNMi Custom Poller | Operations Bridge Analytics uses SMB protocol to mount a CSV data directory on the NNMi system to the OBA Collector host. Because of this mounted data directory, SMB ports must be open. |
| 138 | OBA Collector host | NNM iSPI Performance for Metrics and NNMi Custom Poller | Operations Bridge Analytics uses SMB protocol to mount a CSV data directory on the NNMi system to the OBA Collector host. Because of this mounted data directory, SMB ports must be open. |
| 139 | OBA Collector host | NNM iSPI Performance for Metrics and NNMi Custom Poller | Operations Bridge Analytics uses SMB protocol to mount a CSV data directory on the NNMi system to the OBA Collector host. Because of this mounted data directory, SMB ports must be open. |
| 445 | OBA Collector host | NNM iSPI Performance for Metrics and NNMi Custom Poller | Operations Bridge Analytics uses SMB protocol to mount a CSV data directory on the NNMi system to the OBA Collector host. Because of this mounted data directory, SMB ports must be open. |
| 383 | OBA Application Server and Collector hosts | Operations Agent (OM Performance Agent and Database SPI)" |  |
| 443 or 9000 | ArcSight Connectors, OBA Application Server and Collector hosts | ArcSight Logger | ArcSight Logger can be configured to be accessible on different ports such as 9000. The OBA default for adding a Logger instance is port 443. |
| 1443 or 1521 | OBA Application Server and Collector hosts | Database host used by OM or OMi | 1443 if using MSSQL or PostgreSQL, 1521 if using Oracle. This port might have been changed by the OM or OMi database administrator. |
| 4888 | ArcSight Logger | OBA Collector host | TCP Receiver: This port is used in case log data is collected from the Logger TCP forwarder and not the pull API. In this case the TCP forwarder from the ArcSight Logger source should be able to send data from the Logger server to port 4888. |
| 4447, 9990 | OBA Application Server host |  | JBOSS PORTS |
| 5433 | OBA Application Server and Collector hosts | Vertica | The default Vertica port is 5433. This default value can be changed by the Vertica administrator. |
| 8080 | Web browsers on client devices that access the OBA console | OBA Application Server host | Web browsers connect to the 8080 port using HTTP (non-SSL) to the OBA Application Server host to access the OBA console. (http://<OBA Application Server FQDN> :8080/opsa) |
| 8080 | OBA Application Server and Collector hosts | SiteScope | OBA communicates with SiteScope. This port might have been configured differently by the SiteScope administrator |
| 8089 | OBA Application Server and Collector hosts | Splunk | Used if Splunk is used instead of ArcSight Logger. |
| 8443 (https) 8080 (http) | OBA Application Server and Collector hosts | BSM or OMi Gateway serve | Used for BSM/BPM and OMi integration. |
| 9443 | SiteScope server | OBA Application Server and Collector hosts | This port is the default port of a SiteScope integration instance and can be changed by the SiteScope administrator. |  

Internal traffic is the traffic between OBA Application Server hosts and the OBA Collector hosts. The communication ports shown in "Well-Known Port Mapping (Sources Internal to Operations Analytics)" on page 1 lists the ports used to transmit data among OBA Application Server and Collector host. It works better to disable any firewalls between the OBA Application Server hosts and Operations Bridge Analytics Collector hosts. Each port listed in this table should be opened in both directions (send from it and receive to it).

It works better to disable any firewalls between the OBA Application Server and Collector host.

**Map Ports (Internal Sources to OBA)**

| Port | Open From | Open To | Comments |
|------|-----------|---------|----------|
|381-383| OBA Application Server and Collector host	| OBA Application Server and Collector host | Used by local OM performance agents. |
| 2181 | OBA Application Server and Collector host | OBA Application Server host | Any data flow that uses Apache Zookeeper within Operations Bridge Analytics. |
| 2888, 3888 | OBA Application Server and Collector host | OBA Application Server host | Zookeeper leader election and peer ports. | 
| 4242 | Clients connection to Apache Storm (used internally by Operations Bridge Analytics | OBA Application Server host | Clients connecting to Apache Storm. |
| 6627 | OBA Application Server and Collector host | OBA Application Server host | Apache Storm Nimbus thrift port. |
| 6700-6703 | OBA Application Server and Collector host | OBA Collector hosts | Apache Storm Nimbus worker ports. |
| 9443 | OBA Application Server and Collector host | OBA Collector hosts | Used by the OBA Application Server host to register OBA Collector hosts. |
| 9092 | OBA  Collector host | OBA  Collector hosts | Used by Apache Kafka broker port |

## Installation Steps (ITOM Practitioner Portal)

1. Install Vertica on every node using the Vertica version 8.0.1 delivered with Operations Bridge Analytics. For the installation instructions and information about the prerequisites, see the Vertica installation manual. The documentation includes information about the supported operating systems, known issues, the required kernel parameters, the file system type, multi-node clustering, setting and changing the default password, and so on.
2. Run the following commands on the first node of every cluster: export 

```bash
CONFIG_LOCATION=/opt/vertica/opsa_data 
export DATA_LOCATION=/opt/vertica/opsa_data
export DBADMIN_PWD=<password_dbadmin> 
export instance_name=opsadb 
mkdir -p $DATA_LOCATION $CONFIG_LOCATION /opt/vertica/log 
chown -R dbadmin:verticadba /opt/vertica/log $DATA_LOCATION $CONFIG_LOCATION <br< /opt/vertica/sbin/install_vertica --license CE --accept-eula --dba-user-password $DBADMIN_PWD ----hosts <node1>,<node2>,<node3> 
```
3. On one of the nodes, create the database as the dbadmin user :

```bash
export CONFIG_LOCATION=/opt/vertica/opsa_data export DATA_LOCATION=/opt/vertica/opsa_data export DBADMIN_PWD=<password_dbadmin>  export instance_name=opsadb  admintools -t create_db -D $DATA_LOCATION -c $CONFIG_LOCATION -d $instance_name -p $DBADMIN_PWD -s <node1>,<node2>,<node3> --policy=always Replace <node1>, <node2>, and <node3>
```
with the hostnames of the Vertica cluster nodes.

**Use an Existing Vertica** 

If you want to use an existing Vertica installation, do the following:

Run the following command as the `dbadmin` user to create and start `opsadb`: 
```bash
admintools -t create_db -d opsadb -p dbadmin \--hosts=<VerticaNode1>,<VerticaNode2>,<VerticaNode3> --policy=always
```
You can now use your existing Vertica software. There is no need to install the Vertica application included with OBA.

**Completing Other Steps after Installing Vertica**

**Stabilizing the Vertica Connection when a Firewall is Enabled**

The connection between the Vertica server and the client can be prematurely terminated by a firewall timeout. Examples of these clients with regards to Operations Bridge Analytics involve any connections from either the Operations Bridge Analytics Server or the Operations Bridge Analytics Collector hosts. This could happen when a long-running query is in progress but no data is being passed back to the client, or when the Operations Bridge Analytics internal connection pool is in an idle state and the firewall timeout is less than the `TCP KEEPALIVE` setting on the database server.

**Note: On some Linux distributions, the default `KEEPALIVE` setting is 2 hours or 7200 seconds.**

One possible solution would be to change the `KEEPALIVE` setting to a value lower than the firewall timeout. 

The following command is only an example of how to use the commands, so substitute different values to meet your needs. In the following example, you would run this command on each Vertica node to set the `KEEPALIVE` setting to 10 minutes (`600` seconds): 

```bash
echo 600 > /proc/sys/net/ipv4/tcp_keepalive_time
```

Considering this example, you might do the following on the Vertica server, the Operations Bridge Analytics Server, and the Operations Bridge Analytics Collector host to save these values in the case of these servers resetting. 

**Remember that the values used in the following steps are only there as an example. You must substitute different values to meet your needs:**

* Edit `/etc/sysctl.conf`
   * Add the following three lines:
    ```
    net.ipv4.tcp_keepalive_time = 300 
    net.ipv4.tcp_keepalive_intvl = 60 
    net.ipv4.tcp_keepalive_probes=20

    ```
* save `/etc/sysctl.conf`
* as `root` run `sysctl -p`

**Data Base Load Balancing (if required)**

To enable this native resource load balancing, run the following command as the Vertica `dbadmin` user:

```bash
/opt/vertica/bin/vsql -U dbadmin -c "SELECT SET_LOAD_BALANCE_POLICY('ROUNDROBIN');"
```


## Install OBA application server

You can install the OBA Server software on a supported server. This process installs the Operations Agent 12.01 on the OBA Application Server host. This Operations Agent installation includes permanent licenses for the Operation Agent, Performance Agent (perfd), and Glance software applications. There must not be any version of the Operations Agent pre-installed on the system. If an agent version is already installed, remove it prior to Operations Bridge Analytics installation.

Complete the following steps to deploy Operations Bridge Analytics on a server (OBA Application Server host):

1. `unzip -f HPE_OpsA_3.01_Analytics_Installation.zip`
2. As the `root` user, from the local directory that you placed the run (from the directory to which you extracted the product files.):

```bash
bash ./opsa_3.01_setup.bin
```
3. When prompted, specify that you are installing the Operations Analytics Server.
4. Follow the interactive prompts to complete the installation.

**Installer Troubleshooting**

The OBA Linux installer requires full access to the default temporary directory (the /tmp directory). If this directory is restricted in any way (for example because of security requirements) you should choose a different temporary directory with full access before running the installer.

**How to Change the Installer Working Directory**

You can change the Installer's working directory (by default /tmp) by running the following commands: 
```bash
export IATEMPDIR=/new/tmp 
export _JAVA_OPTIONS=-Djava.io.tmpdir=/new/tmp 
```
where /new/tmp is the new working directory.

## Install OBA Collector

1. Extract the `HPE_OpsA_3.01_Analytics_Installation.zip` to a local directory. Navigate to that directory.
2. As the `root` user, run `bash ./opsa_3.01_setup.bin` from the directory to which you extracted the product files.

**Note:** Do not install OBA from the `/tmp/` directory.