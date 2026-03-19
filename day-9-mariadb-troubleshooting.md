# Day 9: MariaDB Troubleshooting
There is a critical issue going on with the `Nautilus` application in `Stratos DC`. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that mariadb service is down on the database server.

Look into the issue and fix the same.

```bash
[peter@stdb01 ~]$ sudo systemctl status mariadb
× mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: disabled)
     Active: failed (Result: exit-code) since Wed 2025-12-24 15:23:17 UTC; 4min 33s ago
   Duration: 6.413s
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 2690 ExecStartPre=/usr/libexec/mariadb-check-socket (code=exited, status=0/SUCCESS)
    Process: 2724 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir mariadb.service (code=exited, status=1/FAILURE)

Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com mariadb-prepare-db-dir[2724]: Make sure the /var/lib/mysql is empty before running mariadb-prepare-db-dir.
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Child 2724 belongs to mariadb.service.
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Control process exited, code=exited, status=1/FAILURE
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Got final SIGCHLD for state start-pre.
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Failed with result 'exit-code'.
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Service will not restart (restart setting)
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Changed start-pre -> failed
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Job 316 mariadb.service/start finished, result=failed
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: Failed to start MariaDB 10.5 database server.
Dec 24 15:23:17 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Unit entered failed state.
[peter@stdb01 ~]$ ls /var/lib/mysql
ls: cannot access '/var/lib/mysql': No such file or directory
[peter@stdb01 ~]$ sudo mkdir /var/lib/mysql
[peter@stdb01 ~]$ sudo ls -lah /var/lib/mysql
total 12K
drwxr-xr-x 2 root root 4.0K Dec 24 15:28 .
drwxr-xr-x 1 root root 4.0K Dec 24 15:28 ..
[peter@stdb01 ~]$ sudo chown -R mysql:mysql /var/lib/mysql
[peter@stdb01 ~]$ sudo systemctl start mariadb
[peter@stdb01 ~]$ sudo systemctl status mariadb
 mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: disabled)
     Active: active (running) since Wed 2025-12-24 15:30:11 UTC; 18s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 3022 ExecStartPre=/usr/libexec/mariadb-check-socket (code=exited, status=0/SUCCESS)
    Process: 3056 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir mariadb.service (code=exited, status=0/SUCCESS)
    Process: 3269 ExecStartPost=/usr/libexec/mariadb-check-upgrade (code=exited, status=0/SUCCESS)
   Main PID: 3218 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 32 (limit: 411434)
     Memory: 95.4M
     CGroup: /docker/773b5ec9663b910fbf36c4f6526c29dfa8145b4775ef1cdcf1b14cb5183d13c7/system.slice/mariadb.service
             └─3218 /usr/libexec/mariadbd --basedir=/usr

Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[3269]: Remounted /run/systemd/unit-root/run/systemd/incoming.
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[3269]: Remounted /run/systemd/unit-root/run/credentials.
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[3269]: mariadb.service: Executing: /usr/libexec/mariadb-check-upgrade
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Child 3269 belongs to mariadb.service.
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Control process exited, code=exited, status=0/SUCCESS (success)
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Got final SIGCHLD for state start-post.
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Changed start-post -> running
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Job 360 mariadb.service/start finished, result=done
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[1]: Started MariaDB 10.5 database server.
Dec 24 15:30:11 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Failed to send unit change signal for mariadb.service: Connection reset by peer
[peter@stdb01 ~]$ sudo mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.5.27-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.001 sec)

MariaDB [(none)]> exit
Bye
[peter@stdb01 ~]$
```
