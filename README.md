# mariadb-podman-socket-activation

__Status:__ proof of concept experiment

A demo of a templated systemd user service that runs rootless [Podman](https://podman.io)
and starts [MariaDB](https://mariadb.org/) with [systemd socket activation](https://www.freedesktop.org/software/systemd/man/systemd.socket.html).

## Introduction

Characteristics of using socket activation of containers with Podman

* The `podman run` option __--publish__ is not used
* The communication over the established TCP connection will run at native speed.
  (Rootless Podman normally uses slirp4netns which comes with a performance penalty).
  See the [Podman socket activation tutorial](https://github.com/containers/podman/blob/main/docs/tutorials/socket_activation.md#native-network-performance-over-the-socket-activated-socket).
* Possibility to use the `podman run` option __--network=none__ to restrict internet access
  for the container. (A socket-activated TCP socket can still be used by the mariadb container).
  See the [Podman socket activation tutorial](https://github.com/containers/podman/blob/main/docs/tutorials/socket_activation.md#disabling-the-network-with---networknone) and the blog post [_How to limit container privilege with socket activation_](https://www.redhat.com/sysadmin/socket-activation-podman)
* Possibility to use the systemd directive `RestrictAddressFamilies` to restrict general
  internet access for Podman (and its helper programs like conmon and the OCI runtime).
  See the blog post [_How to restrict network access in Podman with systemd_](https://www.redhat.com/sysadmin/podman-systemd-limit-access).
* The source IP address is preserved when using socket activation. In some network configurations
  when using rootless Podman that is not the case. See Podman GitHub [discussion](https://github.com/containers/podman/discussions/10472).

## Requirements

* podman 3.4.4 (or newer)
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
cp -r mariadb*@* ~/.config/systemd/user
```

Run

```
systemctl --user daemon-reload
```

## Usage

### Create a UNIX socket that activates a MariaDB instance

Create a UNIX socket from which the new MariaDB instance _foobar_ will
be started via _socket activation_:

```
systemctl --user start mariadb-unix@foobar.socket
```

Note that there is no need to run `systemctl --user start mariadb-unix@foobar.service`
because of the configured _socket activation_.

Connect to the new MariaDB instance _foobar_
(type `my` as password to log in):

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

### Create a TCP socket that activates a MariaDB instance

Create a TCP socket from which the new MariaDB instance _demo_ will
be started via  _socket activation_:

```
systemctl --user start mariadb-tcp@demo.socket
```

The port number 8090 was specified in
_~/.config/systemd/user/mariadb-tcp@demo.socket.d/override.conf_

Note that there is no need to run `systemctl --user start mariadb-tcp@foobar.service`
because of the configured _socket activation_.

Connect to the new MariaDB instance _demo_
(type `my` as password to log in):

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
