# mariadb-podman-socket-activation

Let's create a templated systemd user service to run rootless Podman
and start MariaDB with socket activation. In this example we will only
make use of UNIX sockets and thus be able to run the container with
__--network=none__.

## Requirements

* podman (version >= ?)

* mariadb client (TODO: try to use a container instead)

## Installation

Create the systemd user configuration directory if it is not already present

```
mkdir -p ~/.config/systemd/user
```

Create the file _~/.config/systemd/user/mariadb@.socket_ with the content

```
[Unit]
Description=MariaDB and Podman with socket activation for %I
[Socket]
ListenStream=%h/mariadb-socket.%i

[Install]
WantedBy=default.target
```

Create the file _~/.config/systemd/user/mariadb@.service_ with the content

```
[Unit]
Description=MariaDB and Podman with socket activation for %I
Wants=network-online.target
After=network-online.target
RequiresMountsFor=%t/containers

[Service]
Environment=PODMAN_SYSTEMD_UNIT=%n-%i
Restart=on-failure
TimeoutStopSec=70
ExecStartPre=/bin/rm -f %t/%n.ctr-id
ExecStartPre=mkdir -p %h/mariadb-data.%i
ExecStart=/usr/bin/podman run \
  --cidfile=%t/%n.ctr-id \
  --cgroups=no-conmon \
  --rm \
  --sdnotify=conmon \
  --replace \
  --name mariadb-%i \
  --detach \
  --volume %h/mariadb-data.%i:/var/lib/mysql:Z \
  --env MARIADB_USER=example-user \
  --security-opt label=disable \
  --pull=never \
  --network none \
  --userns=keep-id \
  --env MARIADB_PASSWORD=my \
  --env MARIADB_ROOT_PASSWORD=my_root \
    docker.io/library/mariadb:latest
ExecStop=/usr/bin/podman stop --ignore --cidfile=%t/%n.ctr-id
ExecStopPost=/usr/bin/podman rm -f --ignore --cidfile=%t/%n.ctr-id
Type=notify
NotifyAccess=all

[Install]
WantedBy=default.target
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

