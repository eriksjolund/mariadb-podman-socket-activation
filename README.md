# mariadb-podman-socket-activation

A demo of a templated systemd user service that runs rootless Podman
and starts MariaDB with socket activation. By only using UNIX sockets
it's possible to run the container with __--network=none__.

## Requirements

* podman (These instructions have been verified to work with __podman 3.4.4__ and __podman 4.0.0-rc4__)

* mariadb client (TODO: try to use a container instead)

## Installation

Clone this repo

```
git clone URL
cd mariadb-podman-socket-activation
```

Create the systemd user configuration directory if it is not already present

```
mkdir -p ~/.config/systemd/user
```

Copy the systemd unit files to _~/.config/systemd/user_

```
cp mariadb@.socket mariadb@.service ~/.config/systemd/user
```

Run

```
systemctl --user daemon-reload
```

## Usage

To create the new MariaDB instance _foobar_

```
systemctl --user start mariadb@foobar.socket
```

To connect to the new MariaDB instance _foobar_
(type `my` as password to log in)
```
$ mariadb --socket ~/mariadb-socket.foobar -p -u example-user
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 10.6.5-MariaDB-1:10.6.5+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> \q
Bye
$ 
```

A new data directory was created for the MariaDB instance
under _~/mariadb-data.foobar/_

```
$ ls -l ~/mariadb-data.foobar/
total 123316
-rw-rw----. 1 esjolund esjolund    417792 Feb  8 18:04 aria_log.00000001
-rw-rw----. 1 esjolund esjolund        52 Feb  8 18:04 aria_log_control
-rw-rw----. 1 esjolund esjolund         9 Feb  8 18:04 ddl_recovery.log
-rw-rw----. 1 esjolund esjolund       946 Feb  8 18:04 ib_buffer_pool
-rw-rw----. 1 esjolund esjolund  12582912 Feb  8 18:04 ibdata1
-rw-rw----. 1 esjolund esjolund 100663296 Feb  8 18:07 ib_logfile0
-rw-rw----. 1 esjolund esjolund  12582912 Feb  8 18:04 ibtmp1
-rw-rw----. 1 esjolund esjolund         0 Feb  8 18:04 multi-master.info
drwx------. 2 esjolund esjolund      4096 Feb  8 18:04 mysql
drwx------. 2 esjolund esjolund        20 Feb  8 18:04 performance_schema
drwx------. 2 esjolund esjolund      8192 Feb  8 18:04 sys
$ 
```
