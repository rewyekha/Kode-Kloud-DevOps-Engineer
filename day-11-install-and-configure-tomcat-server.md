# Day 11: Install and Configure Tomcat Server
The `Nautilus` application development team recently finished the beta version of one of their Java-based applications, which they are planning to deploy on one of the app servers in `Stratos DC`. After an internal team meeting, they have decided to use the `tomcat` application server. Based on the requirements mentioned below complete the task:

a. Install `tomcat` server on `App Server 3`.

b. Configure it to run on port `6300`.

c. There is a `ROOT.war` file on `Jump host` at location `/tmp`.

Deploy it on this tomcat server and make sure the webpage works directly on base URL i.e `curl http://stapp03:6300`

```bash
thor@jumphost ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:MUJC/RQp7AFGREhonhrRl8ZR0FFETLyD3fsU0sUnqlc.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password:
[banner@stapp03 ~]$ sudo yum install -y tomcat-webapps tomcat-admin-webapps

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner:
CentOS Stream 9 - BaseOS                45 kB/s | 7.0 kB     00:00
CentOS Stream 9 - BaseOS                12 MB/s | 8.8 MB     00:00
CentOS Stream 9 - AppStream             41 kB/s | 7.4 kB     00:00
CentOS Stream 9 - AppStream            1.8 MB/s |  26 MB     00:14
CentOS Stream 9 - Extras packages       39 kB/s | 8.0 kB     00:00
CentOS Stream 9 - Extras packages       25 kB/s |  20 kB     00:00
Extra Packages for Enterprise Linux 9  111 kB/s |  24 kB     00:00
Extra Packages for Enterprise Linux 9   19 MB/s |  20 MB     00:01
Extra Packages for Enterprise Linux 9  7.2 kB/s | 993  B     00:00
Extra Packages for Enterprise Linux 9  124 kB/s |  22 kB     00:00
Extra Packages for Enterprise Linux 9  334 kB/s | 259 kB     00:00
Dependencies resolved.
=======================================================================
 Package                  Arch   Version               Repo       Size
=======================================================================
Installing:
 tomcat-admin-webapps     noarch 1:9.0.87-6.el9        appstream  77 k
 tomcat-webapps           noarch 1:9.0.87-6.el9        appstream  80 k
Installing dependencies:
 apr                      x86_64 1.7.0-12.el9          appstream 123 k
 avahi-libs               x86_64 0.8-23.el9            baseos     67 k
 copy-jdk-configs         noarch 4.0-3.el9             appstream  28 k
 cups-libs                x86_64 1:2.3.3op2-36.el9     baseos    260 k
 ecj                      noarch 1:4.20-17.el9         appstream 1.9 M
 freetype                 x86_64 2.10.4-11.el9         baseos    372 k
 graphite2                x86_64 1.3.14-9.el9          baseos     95 k
 harfbuzz                 x86_64 2.7.4-10.el9          baseos    624 k
 java-1.8.0-openjdk-headless
                          x86_64 1:1.8.0.362.b09-4.el9 appstream  32 M
 javapackages-filesystem  noarch 6.4.0-1.el9           appstream  13 k
 javapackages-tools       noarch 6.4.0-1.el9           appstream  34 k
 libbrotli                x86_64 1.0.9-7.el9           baseos    313 k
 libpng                   x86_64 2:1.6.37-12.el9       baseos    117 k
 lksctp-tools             x86_64 1.0.19-2.el9          baseos     94 k
 lua                      x86_64 5.4.4-4.el9           appstream 188 k
 lua-posix                x86_64 35.0-8.el9            appstream 151 k
 nspr                     x86_64 4.36.0-4.el9          appstream 133 k
 nss                      x86_64 3.112.0-4.el9         appstream 722 k
 nss-softokn              x86_64 3.112.0-4.el9         appstream 399 k
 nss-softokn-freebl       x86_64 3.112.0-4.el9         appstream 413 k
 nss-sysinit              x86_64 3.112.0-4.el9         appstream  18 k
 nss-util                 x86_64 3.112.0-4.el9         appstream  88 k
 tomcat                   noarch 1:9.0.87-6.el9        appstream  98 k
 tomcat-el-3.0-api        noarch 1:9.0.87-6.el9        appstream 105 k
 tomcat-jsp-2.3-api       noarch 1:9.0.87-6.el9        appstream  72 k
 tomcat-lib               noarch 1:9.0.87-6.el9        appstream 6.0 M
 tomcat-servlet-4.0-api   noarch 1:9.0.87-6.el9        appstream 284 k
 tzdata-java              noarch 2025c-1.el9           appstream 223 k
Installing weak dependencies:
 tomcat-native            x86_64 1:1.3.0-1.el9         epel       74 k

Transaction Summary
=======================================================================
Install  31 Packages

Total download size: 45 M
Installed size: 134 M
Downloading Packages:
(1/31): avahi-libs-0.8-23.el9.x86_64.r 189 kB/s |  67 kB     00:00
(2/31): cups-libs-2.3.3op2-36.el9.x86_ 581 kB/s | 260 kB     00:00
(3/31): freetype-2.10.4-11.el9.x86_64. 824 kB/s | 372 kB     00:00
(4/31): graphite2-1.3.14-9.el9.x86_64. 264 kB/s |  95 kB     00:00
(5/31): libbrotli-1.0.9-7.el9.x86_64.r 722 kB/s | 313 kB     00:00
(6/31): harfbuzz-2.7.4-10.el9.x86_64.r 1.3 MB/s | 624 kB     00:00
(7/31): apr-1.7.0-12.el9.x86_64.rpm    1.1 MB/s | 123 kB     00:00
(8/31): copy-jdk-configs-4.0-3.el9.noa 1.4 MB/s |  28 kB     00:00
(9/31): libpng-1.6.37-12.el9.x86_64.rp 316 kB/s | 117 kB     00:00
(10/31): ecj-4.20-17.el9.noarch.rpm     21 MB/s | 1.9 MB     00:00
(11/31): javapackages-filesystem-6.4.0 761 kB/s |  13 kB     00:00
(12/31): javapackages-tools-6.4.0-1.el 1.9 MB/s |  34 kB     00:00
(13/31): lua-5.4.4-4.el9.x86_64.rpm    9.9 MB/s | 188 kB     00:00
(14/31): lua-posix-35.0-8.el9.x86_64.r 8.0 MB/s | 151 kB     00:00
(15/31): lksctp-tools-1.0.19-2.el9.x86 264 kB/s |  94 kB     00:00
(16/31): nspr-4.36.0-4.el9.x86_64.rpm  5.4 MB/s | 133 kB     00:00
(17/31): nss-softokn-3.112.0-4.el9.x86  19 MB/s | 399 kB     00:00
(18/31): nss-softokn-freebl-3.112.0-4.  19 MB/s | 413 kB     00:00
(19/31): nss-sysinit-3.112.0-4.el9.x86 1.0 MB/s |  18 kB     00:00
(20/31): nss-util-3.112.0-4.el9.x86_64 4.8 MB/s |  88 kB     00:00
(21/31): tomcat-9.0.87-6.el9.noarch.rp 5.3 MB/s |  98 kB     00:00
(22/31): tomcat-admin-webapps-9.0.87-6 4.2 MB/s |  77 kB     00:00
(23/31): nss-3.112.0-4.el9.x86_64.rpm  5.3 MB/s | 722 kB     00:00
(24/31): tomcat-el-3.0-api-9.0.87-6.el 5.7 MB/s | 105 kB     00:00
(25/31): tomcat-jsp-2.3-api-9.0.87-6.e 4.2 MB/s |  72 kB     00:00
(26/31): tomcat-servlet-4.0-api-9.0.87  13 MB/s | 284 kB     00:00
(27/31): tomcat-webapps-9.0.87-6.el9.n 4.5 MB/s |  80 kB     00:00
(28/31): java-1.8.0-openjdk-headless-1  69 MB/s |  32 MB     00:00
(29/31): tzdata-java-2025c-1.el9.noarc 1.8 MB/s | 223 kB     00:00
(30/31): tomcat-lib-9.0.87-6.el9.noarc  31 MB/s | 6.0 MB     00:00
(31/31): tomcat-native-1.3.0-1.el9.x86 354 kB/s |  74 kB     00:00
-----------------------------------------------------------------------
Total                                   20 MB/s |  45 MB     00:02
Extra Packages for Enterprise Linux 9  1.6 MB/s | 1.6 kB     00:00
Importing GPG key 0x3228467C:
 Userid     : "Fedora (epel9) epel@fedoraproject.org>"
 Fingerprint: FF8A D134 4597 106E CE81 3B91 8A38 72BF 3228 467C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-9
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Running scriptlet: copy-jdk-configs-4.0-3.el9.noarch             1/1
  Running scriptlet: java-1.8.0-openjdk-headless-1:1.8.0.362.b09   1/1
  Preparing        :                                               1/1
  Installing       : javapackages-filesystem-6.4.0-1.el9.noarc    1/31
  Installing       : nspr-4.36.0-4.el9.x86_64                     2/31
  Installing       : nss-util-3.112.0-4.el9.x86_64                3/31
  Installing       : javapackages-tools-6.4.0-1.el9.noarch        4/31
  Installing       : tomcat-el-3.0-api-1:9.0.87-6.el9.noarch      5/31
  Running scriptlet: tomcat-el-3.0-api-1:9.0.87-6.el9.noarch      5/31
  Installing       : tomcat-servlet-4.0-api-1:9.0.87-6.el9.noa    6/31
  Running scriptlet: tomcat-servlet-4.0-api-1:9.0.87-6.el9.noa    6/31
  Installing       : tomcat-jsp-2.3-api-1:9.0.87-6.el9.noarch     7/31
  Running scriptlet: tomcat-jsp-2.3-api-1:9.0.87-6.el9.noarch     7/31
  Installing       : nss-softokn-freebl-3.112.0-4.el9.x86_64      8/31
  Installing       : nss-softokn-3.112.0-4.el9.x86_64             9/31
  Installing       : nss-3.112.0-4.el9.x86_64                    10/31
  Running scriptlet: nss-3.112.0-4.el9.x86_64                    10/31
  Installing       : nss-sysinit-3.112.0-4.el9.x86_64            11/31
  Installing       : tzdata-java-2025c-1.el9.noarch              12/31
  Installing       : lua-posix-35.0-8.el9.x86_64                 13/31
  Installing       : lua-5.4.4-4.el9.x86_64                      14/31
  Installing       : copy-jdk-configs-4.0-3.el9.noarch           15/31
  Installing       : apr-1.7.0-12.el9.x86_64                     16/31
  Installing       : tomcat-native-1:1.3.0-1.el9.x86_64          17/31
  Installing       : lksctp-tools-1.0.19-2.el9.x86_64            18/31
  Installing       : libpng-2:1.6.37-12.el9.x86_64               19/31
  Installing       : libbrotli-1.0.9-7.el9.x86_64                20/31
  Installing       : graphite2-1.3.14-9.el9.x86_64               21/31
  Installing       : harfbuzz-2.7.4-10.el9.x86_64                22/31
  Installing       : freetype-2.10.4-11.el9.x86_64               23/31
  Installing       : avahi-libs-0.8-23.el9.x86_64                24/31
  Installing       : cups-libs-1:2.3.3op2-36.el9.x86_64          25/31
  Installing       : java-1.8.0-openjdk-headless-1:1.8.0.362.b   26/31
  Running scriptlet: java-1.8.0-openjdk-headless-1:1.8.0.362.b   26/31
  Installing       : ecj-1:4.20-17.el9.noarch                    27/31
  Installing       : tomcat-lib-1:9.0.87-6.el9.noarch            28/31
  Running scriptlet: tomcat-1:9.0.87-6.el9.noarch                29/31
  Installing       : tomcat-1:9.0.87-6.el9.noarch                29/31
  Running scriptlet: tomcat-1:9.0.87-6.el9.noarch                29/31
  Installing       : tomcat-admin-webapps-1:9.0.87-6.el9.noarc   30/31
  Installing       : tomcat-webapps-1:9.0.87-6.el9.noarch        31/31
  Running scriptlet: nss-3.112.0-4.el9.x86_64                    31/31
  Running scriptlet: copy-jdk-configs-4.0-3.el9.noarch           31/31
  Running scriptlet: java-1.8.0-openjdk-headless-1:1.8.0.362.b   31/31
  Running scriptlet: tomcat-webapps-1:9.0.87-6.el9.noarch        31/31
  Verifying        : avahi-libs-0.8-23.el9.x86_64                 1/31
  Verifying        : cups-libs-1:2.3.3op2-36.el9.x86_64           2/31
  Verifying        : freetype-2.10.4-11.el9.x86_64                3/31
  Verifying        : graphite2-1.3.14-9.el9.x86_64                4/31
  Verifying        : harfbuzz-2.7.4-10.el9.x86_64                 5/31
  Verifying        : libbrotli-1.0.9-7.el9.x86_64                 6/31
  Verifying        : libpng-2:1.6.37-12.el9.x86_64                7/31
  Verifying        : lksctp-tools-1.0.19-2.el9.x86_64             8/31
  Verifying        : apr-1.7.0-12.el9.x86_64                      9/31
  Verifying        : copy-jdk-configs-4.0-3.el9.noarch           10/31
  Verifying        : ecj-1:4.20-17.el9.noarch                    11/31
  Verifying        : java-1.8.0-openjdk-headless-1:1.8.0.362.b   12/31
  Verifying        : javapackages-filesystem-6.4.0-1.el9.noarc   13/31
  Verifying        : javapackages-tools-6.4.0-1.el9.noarch       14/31
  Verifying        : lua-5.4.4-4.el9.x86_64                      15/31
  Verifying        : lua-posix-35.0-8.el9.x86_64                 16/31
  Verifying        : nspr-4.36.0-4.el9.x86_64                    17/31
  Verifying        : nss-3.112.0-4.el9.x86_64                    18/31
  Verifying        : nss-softokn-3.112.0-4.el9.x86_64            19/31
  Verifying        : nss-softokn-freebl-3.112.0-4.el9.x86_64     20/31
  Verifying        : nss-sysinit-3.112.0-4.el9.x86_64            21/31
  Verifying        : nss-util-3.112.0-4.el9.x86_64               22/31
  Verifying        : tomcat-1:9.0.87-6.el9.noarch                23/31
  Verifying        : tomcat-admin-webapps-1:9.0.87-6.el9.noarc   24/31
  Verifying        : tomcat-el-3.0-api-1:9.0.87-6.el9.noarch     25/31
  Verifying        : tomcat-jsp-2.3-api-1:9.0.87-6.el9.noarch    26/31
  Verifying        : tomcat-lib-1:9.0.87-6.el9.noarch            27/31
  Verifying        : tomcat-servlet-4.0-api-1:9.0.87-6.el9.noa   28/31
  Verifying        : tomcat-webapps-1:9.0.87-6.el9.noarch        29/31
  Verifying        : tzdata-java-2025c-1.el9.noarch              30/31
  Verifying        : tomcat-native-1:1.3.0-1.el9.x86_64          31/31

Installed:
  apr-1.7.0-12.el9.x86_64
  avahi-libs-0.8-23.el9.x86_64
  copy-jdk-configs-4.0-3.el9.noarch
  cups-libs-1:2.3.3op2-36.el9.x86_64
  ecj-1:4.20-17.el9.noarch
  freetype-2.10.4-11.el9.x86_64
  graphite2-1.3.14-9.el9.x86_64
  harfbuzz-2.7.4-10.el9.x86_64
  java-1.8.0-openjdk-headless-1:1.8.0.362.b09-4.el9.x86_64
  javapackages-filesystem-6.4.0-1.el9.noarch
  javapackages-tools-6.4.0-1.el9.noarch
  libbrotli-1.0.9-7.el9.x86_64
  libpng-2:1.6.37-12.el9.x86_64
  lksctp-tools-1.0.19-2.el9.x86_64
  lua-5.4.4-4.el9.x86_64
  lua-posix-35.0-8.el9.x86_64
  nspr-4.36.0-4.el9.x86_64
  nss-3.112.0-4.el9.x86_64
  nss-softokn-3.112.0-4.el9.x86_64
  nss-softokn-freebl-3.112.0-4.el9.x86_64
  nss-sysinit-3.112.0-4.el9.x86_64
  nss-util-3.112.0-4.el9.x86_64
  tomcat-1:9.0.87-6.el9.noarch
  tomcat-admin-webapps-1:9.0.87-6.el9.noarch
  tomcat-el-3.0-api-1:9.0.87-6.el9.noarch
  tomcat-jsp-2.3-api-1:9.0.87-6.el9.noarch
  tomcat-lib-1:9.0.87-6.el9.noarch
  tomcat-native-1:1.3.0-1.el9.x86_64
  tomcat-servlet-4.0-api-1:9.0.87-6.el9.noarch
  tomcat-webapps-1:9.0.87-6.el9.noarch
  tzdata-java-2025c-1.el9.noarch

Complete!
[banner@stapp03 ~]$ sudo vi /etc/tomcat/server.xml
[banner@stapp03 ~]$ sudo systemctl restart tomcat
[sudo] password for banner:
[banner@stapp03 ~]$ sudo systemctl start tomcat
sudo systemctl enable tomcat
Created symlink /etc/systemd/system/multi-user.target.wants/tomcat.service → /usr/lib/systemd/system/tomcat.service.
[banner@stapp03 ~]$ sudo systemctl status tomcat
 tomcat.service - Apache Tomcat Web Application Container
     Loaded: loaded (/usr/lib/systemd/system/tomcat.service; enabled; preset: disabled)
     Active: active (running) since Sat 2025-12-27 06:22:51 UTC; 20s ago
   Main PID: 1569 (java)
      Tasks: 50 (limit: 411434)
     Memory: 137.8M
     CGroup: /docker/8a3103b6092b06b2386a6e5c00cd5e3b44d7185e8a0845c9472786e9b5ca0066/system.slice/tomcat.service
             └─1569 /usr/lib/jvm/jre/bin/java -Djavax.sql.DataSource.Factory=org.apache.commons.dbcp.BasicDataSourceFactory -classpath /usr/share/tomcat/bin/bootstrap.jar:/usr/share/tomcat/bin/tomcat-juli.jar: -Dcatalina.base=/usr/share/tomcat -Dcatalina.home=/usr/share/tomcat -Djava.endorsed.dirs= -Djava.io.tmpdir=/var/cache/tomcat/temp -Djava.util.logging.config.file=/usr/share/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Dsun.io.useCanonCaches=false org.apache.catalina.startup.Bootstrap start

Dec 27 06:22:56 stapp03.stratos.xfusioncorp.com server[1569]: 27-Dec-2025 06:22:56.143 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-6300"]
Dec 27 06:22:56 stapp03.stratos.xfusioncorp.com server[1569]: 27-Dec-2025 06:22:56.219 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [2909] milliseconds
Dec 27 06:23:03 stapp03.stratos.xfusioncorp.com systemd[1]: tomcat.service: Trying to enqueue job tomcat.service/start/replace
Dec 27 06:23:03 stapp03.stratos.xfusioncorp.com systemd[1]: tomcat.service: Installed new job tomcat.service/start as 66
Dec 27 06:23:03 stapp03.stratos.xfusioncorp.com systemd[1]: tomcat.service: Enqueued job tomcat.service/start as 66
Dec 27 06:23:03 stapp03.stratos.xfusioncorp.com systemd[1]: tomcat.service: Job 66 tomcat.service/start finished, result=done
Dec 27 06:23:03 stapp03.stratos.xfusioncorp.com systemd[1]: tomcat.service: Changed dead -> running
Dec 27 06:23:03 stapp03.stratos.xfusioncorp.com systemd[1]: tomcat.service: Failed to reset devices.allow/devices.deny: Operation not permitted
Dec 27 06:23:04 stapp03.stratos.xfusioncorp.com systemd[1]: tomcat.service: Failed to set 'trusted.invocation_id' xattr on control group /docker/8a3103b6092b06b2386a6e5c00cd5e3b44d7185e8a0845c9472786e9b5ca0066/system.slice/tomcat.service, ignoring: Operation not permitted
Dec 27 06:23:04 stapp03.stratos.xfusioncorp.com systemd[1]: tomcat.service: Failed to remove 'trusted.delegate' xattr flag on control group /docker/8a3103b6092b06b2386a6e5c00cd5e3b44d7185e8a0845c9472786e9b5ca0066/system.slice/tomcat.service, ignoring: Operation not permitted
[banner@stapp03 ~]$ scp /tmp/ROOT.war banner@stapp03:/tmp/
The authenticity of host 'stapp03 (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:MUJC/RQp7AFGREhonhrRl8ZR0FFETLyD3fsU0sUnqlc.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password:
stat local "/tmp/ROOT.war": No such file or directory
[banner@stapp03 ~]$ exit
logout
Connection to stapp03 closed.
thor@jumphost ~$ ls -l /tmp/ROOT.war
-rw-r--r-- 1 root root 4529 Dec 27 06:02 /tmp/ROOT.war
thor@jumphost ~$ scp /tmp/ROOT.war banner@stapp03:/tmp/
banner@stapp03's password:
ROOT.war                               100% 4529    12.1MB/s   00:00
thor@jumphost ~$ ssh banner@stapp03
banner@stapp03's password:
Last login: Sat Dec 27 06:15:05 2025 from 172.16.238.2
[banner@stapp03 ~]$ sudo mv /tmp/ROOT.war /var/lib/tomcat/webapps/
[sudo] password for banner:
[banner@stapp03 ~]$ sudo chown tomcat:tomcat /var/lib/tomcat/webapps/ROOT.war
[banner@stapp03 ~]$ sudo systemctl restart tomcat
[banner@stapp03 ~]$ curl http://localhost:6300
!DOCTYPE html>
!--
To change this license header, choose License Headers in Project Properties.
To change this template file, choose Tools | Templates
and open the template in the editor.
-->
html>
    head>
        title>SampleWebApp/title>
        meta charset="UTF-8">
        meta name="viewport" content="width=device-width, initial-scale=1.0">
    /head>
    body>
        h2>Welcome to xFusionCorp Industries!/h2>
        br>

    /body>
/html>
[banner@stapp03 ~]$ curl http://stapp03:6300
!DOCTYPE html>
!--
To change this license header, choose License Headers in Project Properties.
To change this template file, choose Tools | Templates
and open the template in the editor.
-->
html>
    head>
        title>SampleWebApp/title>
        meta charset="UTF-8">
        meta name="viewport" content="width=device-width, initial-scale=1.0">
    /head>
    body>
        h2>Welcome to xFusionCorp Industries!/h2>
        br>

    /body>
/html>
[banner@stapp03 ~]$
```

### Task Summary
* Install **Tomcat** on **App Server 3 (stapp03)**
* Configure Tomcat to run on **port 6300**
* Deploy **ROOT.war** from **Jump Host (/tmp)** so it works on
   `curl http://stapp03:6300`

***

### 1 SSH to App Server 3
From **jump host**:

```bash
ssh banner@stapp03
```

Password:

```
BigGr33n
```

***

### 2 Install Tomcat
```bash
sudo yum install -y tomcat tomcat-webapps tomcat-admin-webapps
```

***

### 3 Change Tomcat Port to 6300
Edit server configuration:

```bash
sudo vi /etc/tomcat/server.xml
```

Find:

```xml
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol"
```

Change **8080 → 6300**:

```xml
<Connector port="6300" protocol="org.apache.coyote.http11.Http11NioProtocol"
```

Save and exit.

***

### 4 Start & Enable Tomcat
```bash
sudo systemctl start tomcat
sudo systemctl enable tomcat
```

Verify:

```bash
sudo systemctl status tomcat
```

***

### 5 Copy ROOT.war from Jump Host
Exit to jump host if needed, then:

```bash
scp /tmp/ROOT.war banner@stapp03:/tmp/
```

***

### 6 Deploy ROOT.war
On **stapp03**:

```bash
sudo mv /tmp/ROOT.war /var/lib/tomcat/webapps/
sudo chown tomcat:tomcat /var/lib/tomcat/webapps/ROOT.war
```

Restart Tomcat to deploy:

```bash
sudo systemctl restart tomcat
```

***

### 7 Verify Deployment
On **stapp03**:

```bash
curl http://localhost:6300
```

Or from jump host:

```bash
curl http://stapp03:6300
```

Page should load successfully on **base URL** (no `/ROOT`).

***

### Final Checklist
* Tomcat installed
* Running on port **6300**
* ROOT.war deployed
* Application accessible at `http://stapp03:6300`
