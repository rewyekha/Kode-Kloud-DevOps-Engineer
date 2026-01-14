# Day 17: Install and Configure PostgreSQL

The `Nautilus` application development team has shared that they are planning to deploy one newly developed application on `Nautilus` infra in `Stratos DC`. The application uses PostgreSQL database, so as a pre-requisite we need to set up PostgreSQL database server as per requirements shared below:

PostgreSQL database server is already installed on the `Nautilus` database server.

a. Create a database user `kodekloud_gem` and set its password to `BruCStnMT5`.

b. Create a database `kodekloud_db7` and grant full permissions to user `kodekloud_gem` on this database.

`Note:` Please do not try to restart PostgreSQL server service.

```
thor@jumphost ~$ ssh peter@stdb01.stratos.xfusioncorp.com
The authenticity of host 'stdb01.stratos.xfusioncorp.com (172.17.0.4)' can't be established.
ED25519 key fingerprint is SHA256:0zEF5TmdFJd42RJ6v1kDSSgHmbXE5gXLJFJWMu39V3Q.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stdb01.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
peter@stdb01.stratos.xfusioncorp.com's password: 
[peter@stdb01 ~]$ 
[peter@stdb01 ~]$ sudo su - postgres

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for peter: 
[postgres@stdb01 ~]$ 
[postgres@stdb01 ~]$ psql
psql (13.14)
Type "help" for help.

postgres=# CREATE USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';
CREATE ROLE
postgres=# CREATE DATABASE kodekloud_db7;
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE kodekloud_db7 TO kodekloud_gem;
GRANT
postgres=# \q
[postgres@stdb01 ~]$ psql -U kodekloud_gem -d kodekloud_db7 -h localhost
Password for user kodekloud_gem: 
psql (13.14)
Type "help" for help.

kodekloud_db7=> /q
kodekloud_db7-> /q
kodekloud_db7-> exit
Use \q to quit.
kodekloud_db7-> 
```
