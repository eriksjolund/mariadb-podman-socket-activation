# mariadb-podman-socket-activation

__Status:__ proof of concept experiment

A demo of a templated systemd user service that runs rootless Podman
and starts MariaDB with socket activation.

Interestingly, it's possible to use `podman run --network=none ...`
with both UNIX sockets and TCP sockets, when using socket activation.

Overview:
| socket type | __--security-opt label=__ | systemd drop-in configuration file |
| --          | --                        | --                            |
| TCP socket  | enable (the default)      | required to specify TCP port number |
| UNIX socket | disable       | not required (use the `%i` [specifier](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#Specifiers) to expand the instance name in the UNIX socket path) |

Support for socket activation was added to MariaDB in release __10.6__ (released April 2021).

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
cp -r mariadb@.* ~/.config/systemd/user
```

Run

```
systemctl --user daemon-reload
```

## Usage

### Create a new MariaDB instance listening on a UNIX socket

To create the new MariaDB instance _foobar_

```
systemctl --user start mariadb-unix@foobar.socket
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
The MariaDB instance used the bind-mounted directory _~/mariadb-data-unix.foobar/_
to store its data:

```
$ ls -l ~/mariadb-data-unix.foobar/
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

### Create a new MariaDB instance listening on a TCP socket

To create the new MariaDB instance _demo_ that listens
on the TCP port 8090

```
systemctl --user start mariadb-tcp@demo.socket
```

The port number 8090 was specified in
_~/.config/systemd/user/mariadb-tcp@demo.socket.d/override.conf_

To connect to the new MariaDB instance _demo_
(type `my` as password to log in)
```
$ mariadb -h 127.0.0.1 --port 8090 -p -u example-user
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

The MariaDB instance used the bind-mounted directory _~/mariadb-data-tcp.demo/_
to store its data:

```
$ ls -l ~/mariadb-data-tcp.demo/
total 123316
-rw-rw----. 1 esjolund esjolund    417792 Feb  8 21:13 aria_log.00000001
-rw-rw----. 1 esjolund esjolund        52 Feb  8 21:13 aria_log_control
-rw-rw----. 1 esjolund esjolund         9 Feb  8 21:13 ddl_recovery.log
-rw-rw----. 1 esjolund esjolund       946 Feb  8 21:13 ib_buffer_pool
-rw-rw----. 1 esjolund esjolund  12582912 Feb  8 21:13 ibdata1
-rw-rw----. 1 esjolund esjolund 100663296 Feb  8 21:17 ib_logfile0
-rw-rw----. 1 esjolund esjolund  12582912 Feb  8 21:13 ibtmp1
-rw-rw----. 1 esjolund esjolund         0 Feb  8 21:13 multi-master.info
drwx------. 2 esjolund esjolund      4096 Feb  8 21:13 mysql
drwx------. 2 esjolund esjolund        20 Feb  8 21:13 performance_schema
drwx------. 2 esjolund esjolund      8192 Feb  8 21:13 sys
$ 
```
