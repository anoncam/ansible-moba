# Manual Install Notes

c4.4xlarge instance using fresh RHEL OS. `(ami-92a5e3f3`).

localhost is deployed as the node, but Vertica is designed for a minimum of three nodes.



```bash
# Opinionated reccommendation on the terminal text editor 

curl https://getmic.ro | bash -

chmod +x ./micro

mv micro /usr/bin

# Now replace 'vi' or 'nano' with 'micro'
```

```bash
# ssh as ec2-user

# be root
sudo su -

# update yum
yum update -y

# curl the binaries from Nexus
 
yum install unzip zip dialog libX11 -y
cd /home/ec2-user
unzip *Vertica*
cd <directory you just created from unzip>

rpm -i vertica-8*.rpm

```

```bash
#  Use the dd command to create a swap file on the root file system, where "bs" is the block size and "count" is the number of blocks.
$ sudo dd if=/dev/zero of=/swapfile bs=1G count=4

# Set permissions to swapfile
$ chmod 600 /swapfile

# Set up a Linux swap area: 
$ mkswap /swapfile

# Make the swap file available for immediate use by adding the swap file to swap space: 
$ swapon /swapfile

# Verify that the procedure was successful: 
$ swapon -s

# make the swap persistent 
$ micro /etc/fstab # run as root and Ctrl + S to save
# add this to the last line: /swapfile swap swap defaults 0 0
```
Add the following environmental variables to `/etc/bashrc` 
Make sure they are loaded at the time of installation.


```bash
export CONFIG_LOCATION=/opt/vertica/opsa_data

export DATA_LOCATION=/opt/vertica/opsa_data

export DBADMIN_PWD=vertica

export instance_name=opsadb

mkdir -p $DATA_LOCATION $CONFIG_LOCATION /opt/vertica/log

chown -R dbadmin:verticadba /opt/vertica/log $DATA_LOCATION $CONFIG_LOCATION


/opt/vertica/sbin/install_vertica --license CE --accept-eula --dba-user-password $DBADMIN_PWD --hosts 127.0.0.1 --failure-threshold FAIL

```

```bash
Vertica Analytic Database 8.1.1-0 Installation Tool
AWS Detected. Using AWS defaults.
   AWS Default: --data-dir was not specified, using default: /vertica/data
   AWS Default: --dba-user-password-disabled was not specified,  disabling dba password by default while on AWS
   AWS Default: --point-to-point was not specified,  enabling point-to-point spread communication by default while on AWS
> Validating options...
Mapping hostnames in --hosts (-s) to addresses...
> Starting installation tasks.
> Getting system information for cluster (this may take a while)...
Default shell on nodes:
127.0.0.1 /bin/bash
> Validating software versions (rpm or deb)...
> Beginning new cluster creation...
successfully backed up admintools.conf on 127.0.0.1
> Creating or validating DB Admin user/group...
Successful on hosts (1): 127.0.0.1
   Provided DB Admin account details: user = dbadmin, group = verticadba, home = /home/dbadmin
   Creating group... Group already exists
   Validating group... Okay
   Creating user... User already exists
   Validating user... Okay
> Validating node and cluster prerequisites...
Prerequisites not fully met during local (OS) configuration for
verify-127.0.0.1.xml:
   HINT (S0305): https://my.vertica.com/docs/8.1.x/HTML/index.htm#cshid=S0305
       TZ is unset for dbadmin. Consider updating .profile or .bashrc
   WARN (S0160): https://my.vertica.com/docs/8.1.x/HTML/index.htm#cshid=S0160
       These disks do not have 'ext3' or 'ext4' filesystems: '/dev/xvda2' =
       'xfs'
System prerequisites passed.  Threshold = FAIL
> Establishing DB Admin SSH connectivity...
Installing/Repairing SSH keys for dbadmin
> Setting up each node and modifying cluster...
Creating Vertica Data Directory...
Updating agent...
> Sending new cluster configuration to all nodes...
Starting agent...
> Completing installation...
Running upgrade logic
No spread upgrade required: /opt/vertica/config/vspread.conf not found on any node
Installation complete.
```

**When installing locally, the file `/opt/vertica/config/local-spread.conf` needs to be configured to the system parameters.**

The file `/opt/vertica/config/admintools.conf` needs to be edited to the system parameters as well.  An example `.conf` file looks like:

```conf
[dbadmin@ip-172-31-47-6 opsa_data]$ cat  /opt/vertica/config/admintools.conf
[Configuration]
last_port = 5433
tmp_dir = /tmp
default_base = /home/dbadmin
format = 3
install_opts = --license CE --accept-eula --dba-user-password '*******' --hosts '127.0.0.1' --failure-threshold FAIL
spreadlog = False
atdebug = False
controlsubnet = default
ipv6 = False
controlmode = pt2pt
unreachable_host_caching = True
admintools_config_version = 103

[Cluster]
hosts = 127.0.0.1

[Nodes]
node0001 = 127.0.0.1,/vertica/data,/vertica/data

[SSHConfig]
ssh_user =
ssh_ident =
ssh_options = -oConnectTimeout=30 -o TCPKeepAlive=no -o ServerAliveInterval=15 -o ServerAliveCountMax=2 -o StrictHostKeyChecking=no -o BatchMode=yes
```

## Log
```
[07/23/19]

[root@ip-172-31-47-6 Analytics_Installation]# ./*bin
Preparing to install
Extracting the JRE from the installer archive...
Unpacking the JRE...
Extracting the installation resources from the installer archive...
Configuring the installer for this system's environment...

Launching installer...


Graphical installers are not supported by the VM. The console mode will be used instead...

===============================================================================
Operations Bridge Analytics                      (created with InstallAnywhere)
-------------------------------------------------------------------------------

Preparing CONSOLE Mode Installation...




===============================================================================
Micro Focus Installer
---------------------


PRESS <ENTER> TO CONTINUE:
Please wait while Micro Focus Installer is initializing ...

Introduction
------------

Welcome to the installation for Operations Bridge Analytics 03.04
Micro Focus Installer will guide you through the installation. It is strongly
recommended that you quit all programs before continuing with this
installation.

Application Media Location : /home/ec2-user/OneDrive_1_7-19-2019/OBA_3.04_Anal
ytics_Installation/Analytics_Installation/packages/
Installation Log File : /tmp/MicroFocusOvInstaller/opsa_03.04/opsa_03.04_2019.
07.24_00_24_MicroFocusOvInstallerLog.txt
Respond to each prompt to proceed to the next step in the installation.
You may cancel this installation at any time by typing 'quit'.

PRESS <ENTER> TO CONTINUE:


 Copyright 2015-2017 EntIT Software LLC



I accept the terms of the License Agreement (Y/N): Y


Install Groups are combined sets of features.
If you want to change something on a previous step, type 'back'.
You may cancel this installation at any time by typing 'quit'.



  ->1- Operations Bridge Analytics Server: ()
    2- Operations Bridge Analytics Collector: ()

Please select one of the above groups ...:


===============================================================================
Install Requirements Checks
---------------------------

==============================================================================
=
 Verifying : Verifying free disk space ...  [Completed]
 Verifying : Validating Media path...  [Completed]
 Verifying : Validate OS  [Completed]
 Verifying : Checking free disk space...  [Completed]
 Verifying : Firewall Turn Off  [Completed]
 Verifying : IP Check  [Completed]
 Verifying : SHR Prerequisites Validation...  [Completed]
 Verifying : Validating RPM Prerequisites  [Completed]
==============================================================================
=
Performing checks ...
Details :  performing checks ... please wait
Executing initialize action :
Executing initialize action :
Executing initialize action :
Executing initialize action :
Executing initialize action :
Executing initialize action :
Executing initialize action :
[Failed]: ERROR: Missing some prerequisites RPMs.
One or more above checks failed!
If you want to change something on a previous step, type 'back'.
You may cancel this installation at any time by typing 'quit'.

==============================================================================
=
 Verifying : Verifying free disk space ...  [Passed]
 Verifying : Validating Media path...  [Passed]
 Verifying : Validate OS  [Passed]
 Verifying : Checking free disk space...  [Passed]
 Verifying : Firewall Turn Off  [Passed]
 Verifying : IP Check  [Passed]
 Verifying : SHR Prerequisites Validation...  [Passed]
 Verifying : Validating RPM Prerequisites  [Failed]
==============================================================================
=
Please press any key to perform checks again or 'q' to exit:


```
## Considerations

```sql
[root@ip-172-31-47-6 AWS]# cat ddl/install.sql
/*****************************
 * Vertica Analytic Database
 *
 * Install AWS Library
 *
 * Copyright (c) 2005 - 2016, Hewlett-Packard Development Co., L.P.
 */


-- Step 1: Create LIBRARY
\set aws_libfile '\'/opt/vertica/packages/AWS/lib/aws.so\'';
CREATE OR REPLACE LIBRARY awslib AS :aws_libfile;

-- Step 2: Create Functions
CREATE OR REPLACE FUNCTION aws_set_config AS
LANGUAGE 'C++' NAME 'AwsSetConfigFactory' LIBRARY awslib NOT FENCED;

CREATE OR REPLACE FUNCTION aws_get_config AS
LANGUAGE 'C++' NAME 'AwsGetConfigFactory' LIBRARY awslib NOT FENCED;

CREATE OR REPLACE SOURCE s3 AS
LANGUAGE 'C++' NAME 'S3SourceFactory' LIBRARY awslib NOT FENCED;

CREATE OR REPLACE TRANSFORM FUNCTION s3export AS
LANGUAGE 'C++' NAME 'S3ExportFactory' LIBRARY awslib NOT FENCED;

CREATE OR REPLACE TRANSFORM FUNCTION s3export_partition AS
LANGUAGE 'C++' NAME 'S3ExportPartitionFactory' LIBRARY awslib NOT FENCED;
```
