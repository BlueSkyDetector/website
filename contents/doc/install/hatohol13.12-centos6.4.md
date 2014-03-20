How to install Hatohol on CentOS 6.4 (x86_64) with RPM files
============================================================

[Japanese](hatohol13.12-centos6.4-ja.md)

Installation of needed packages
-------------------------------
### json-glib
Download the RPM from the following URL.

- https://github.com/project-hatohol/json-glib-for-distribution/raw/master/RPMS/x86_64/json-glib-0.12.6-1PH.x86_64.rpm

Install the package as

    # rpm -Uhv json-glib-0.12.6-1PH.x86_64.rpm

### bootstrap
Download the RPM from the following URL.

- https://github.com/project-hatohol/bootstrap-for-hatohol/raw/master/RPMS/x86_64/bootstrap-for-hatohol-2.3.2-1PH.x86_64.rpm

Install the package as

    # rpm -Uhv bootstrap-for-hatohol-2.3.2-1PH.x86_64.rpm

### Django
Download the RPM from the following URL.

- https://github.com/project-hatohol/Django-for-distribution/raw/master/dist/Django-1.5.3-1.noarch.rpm

Install the package as

    # rpm -Uhv Django-1.5.3-1.noarch.rpm


### Dependent packages 
These packages are needed for hatohol.
- libsoup
- libuuid
- MySQL
- MySQL-python
- mod_wsgi
- httpd

Install these packages as

    # yum install libsoup libuuid mysql MySQL-python mod_wsgi httpd



### Hatohol Server
Download the RPM from the following URL.

- https://github.com/project-hatohol/hatohol-packages/raw/master/RPMS/13.12/hatohol-13.12-1.el6.x86_64.rpm

Install the package as

    # rpm -Uhv hatohol-13.12-1.el6.x86_64.rpm

### Hatohol Client
Download the RPM from the following URL.

- https://github.com/project-hatohol/hatohol-packages/raw/master/RPMS/13.12/hatohol-client-13.12-1.el6.x86_64.rpm

Install the package as

    # hatohol-client-13.12-1.el6.x86_64.rpm

Setup
-----
### Installation of MySQL server
If you already have MySQL server and use it, you can skip this step.

    # yum install mysql-server
    # chkconfig mysqld on
    # service mysqld start

### Setup of Hatohol cache DB

You have to prepare a directory for Hatohol cache DB. Here we use '/var/lib/hatohol' as an example.

Make the directory if needed.

    # mkdir /var/lib/hatohol

For using the above directory, you have to make '/etc/hatohol/initrc'.    
And write environmental variables 'HATOHOL_DB_DIR' as below.

    export HATOHOL_DB_DIR=/var/lib/hatohol

### Initialization of Hatohol DB

Copy the configuration template file to an arbitrary directory.

    # cp /usr/share/hatohol/hatohol-config.dat.example ~/hatohol-config.dat

Add the information of ZABBIX server and nagios server in the copied configuration file.
Some rules and examples are in the file as comments.

Reflect the configuration to the DB as

    # hatohol-config-db-creator hatohol-config.dat

### Setup of Hatohol Client

- Setup of MySQL Database for Hatohol Client.

After you login to MySQL, execute commands as below.

    > CREATE DATABASE hatohol_client;
    > GRANT ALL PRIVILEGES ON hatohol_client.* TO hatohol@localhost IDENTIFIED BY 'hatohol';

- Add the table in Hatohol Client DB.

Execute command as below.

    # /usr/libexec/hatohol/client/manage.py syncdb
    
### Start of Hatohol Server

    # service hatohol start

When Hatohol server successfully starts, you can see log messages such as the following in /var/log/messages.

    Oct  8 09:46:58 localhost hatohol[3038]: [INFO] <DBClientConfig.cc:336> Configuration DB Server: localhost, port: (default), DB: hatohol, User: hatohol, use password: yes
    Oct  8 09:46:58 localhost hatohol[3038]: [INFO] <main.cc:165> started hatohol server: ver. 13.12
    Oct  8 09:46:58 localhost hatohol[3038]: [INFO] <FaceRest.cc:121> started face-rest, port: 33194
    Oct  8 09:46:59 localhost hatohol[3038]: [INFO] <ArmZabbixAPI.cc:925> started: ArmZabbixAPI (server: testZbxSv1)
    Oct  8 09:47:01 localhost hatohol[3038]: [INFO] <ArmZabbixAPI.cc:925> started: ArmZabbixAPI (server: testZbxSv2)

> ** TROUBLE SHOOT ** Currenty, Hatohol Server generates all status logs into syslog as USER.INFO. By default settings, USER.INFO is routed to /var/log/messages in CentOS 6.

### Start of Hatohol Client

    # service httpd start

Access with a web browser
-------------------------
### Check of the setting of iptables and SELinux
By default, some security mechanisms such as SELinux and iptables block the access from other computers.
You have to deactivate them if needed.
> ** WARNING **
> You should do the following things after you understand the security risk.

You can confirm the current SELinux status as

    # getenforce
    Enforcing

If 'Enforcing' is replied, it is enabled. And you can disable it as

    # setenforce 0
    # getenforce
    Permissive

> ** Tips **
> By editing /etc/selinux/config, it can be disabled permanently.

As for iptables, an allowed port can be added by editing /etc/sysconfig/iptables.
The following is an example to allow port 8000.

      -A INPUT -p icmp -j ACCEPT
      -A INPUT -i lo -j ACCEPT
      -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
     +-A INPUT -m state --state NEW -p tcp --dport 8000 -j ACCEPT
      -A INPUT -j REJECT --reject-with icmp-host-prohibited
      -A FORWARD -j REJECT --reject-with icmp-host-prohibited

> ** NOTE ** The mark '+' at the head means a newly added line.

Then, the following command reloads the iptables setting.

    # service iptables restart

### View of Hatohol information
For example, if the Hatohol client runs on computer: 192.168.1.1,
Open the following URL from your Browser.

- http://192.168.1.1/viewer/

> ** Note **
> Currently the above pages have been checked with Google Chrome and Firefox.
> When using Internet Explorer, display layouts may collapse depending on the version of use.
