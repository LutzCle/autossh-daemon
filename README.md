# autossh on FreeBSD

This is a guide to setup autossh-daemon, a small script to automatically run
autossh as a reverse SSH tunnel on FreeBSD.

We assume three hosts: a user device (e.g., a laptop), `$proxy_host`, and
`$remote_host`. After following the guide, it will be possible to ssh from user
device to `$remotehost` via `$proxy_host`. Substitute the variables with actual
hostnames as required.

## Setup guide on proxy host

Create a new user account called `autossh`, and make it accessible via SSH.

## Setup guide on remote host

```sh
# Install autossh
pkg install autossh

# Edit autossh-daemon file to configure $proxy_host
vim autossh

# Install autossh-daemon file
install -o root -g wheel -m 755 autossh $proxy_host:/usr/local/etc/rc.d/autossh

# Create user
pw useradd -n autossh -g autossh -u 2223 -c 'The autossh daemon' -s /usr/bin/nologin -w no

# Generate ssh key
su -m autossh -c '/usr/bin/ssh-keygen -t ecdsa'

# Copy public key to remote host
cat ~autossh/.ssh/id_ecdsa.pub | ssh $proxy_host tee --append ~autossh/.ssh/authorized_keys

# Login manually to create ~autossh/.ssh/known_hosts
su -m autossh -c '/usr/bin/ssh autossh@$proxy_host'

/usr/local/etc/rc.d/autossh enable
/usr/local/etc/rc.d/autossh start
```

## Setup guide on user device

```sh
cat >> ~/.ssh/config <<EOF
Host $remote_host
    Hostname $proxy_host
    Port 7000
    ProxyJump $proxy_host
EOF
```

## SSH into remote host from user device

```sh
ssh $remote_host
```
