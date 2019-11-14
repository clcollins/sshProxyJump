# Connecting to remote hosts though a proxy or bastion with ssh ProxyJump

The concept of [bastion hosts](https://en.wikipedia.org/wiki/Bastion_host) is nothing new to computing. Baston hosts are usually public-facing, hardened systems that serve as an entrypoint to systems behind a firewall or other restricted location, and is especially popular with the rise of cloud computing. 

The ssh command has an easy way to make use of bastion hosts to connect to a remote host with a single command.  Instead of first ssh-ing to the bastion host, and then using ssh on the bastion to connect to the remote host, ssh can create the initial and second connections itself, using `ProxyJump`.


## ProxyJump

The `ProxyJump`, or `-J` flag, was introduced in ssh version 7.3.  To use it, simply specify the bastion host to connect through after the `-J` flag, and the remote host 

```sh
ssh -J bastion-host remote-host
```

You can also set specific usernames and ports, if they differ between the hosts:

```sh
ssh -J user@bastion:port user@remote:port
```

The ssh man (or manual) page (`man ssh`) notes that multiple, comma-separated hostnames can be specified to jump through a series of hosts.

```sh
ssh -J bastion1,bastion2 remote
```

This would be useful if there were multiple levels of separation between a bastion and a final remote host.  For example, a public bastion host giving access to a "web tier" set of hosts, from which a further protected "database tier" group might be accessed.

## Hard-coding proxy hosts in ~/.ssh/config

The `-J` flag gives the flexibiltiy to easily specify proxy and remote hosts as needed, but if specific bastion host is regularly used to connect to a specific remote host, the `ProxyJump` configuration can be set in `~/.ssh/config` to automatically make the connection to the bastion en-route to the remote host.

```txt
### The Bastion Host
Host bastion-host-nickname
  HostName bastion-hostname

### The Remote Host
Host remote-host-nickname
  HostName remote-hostname
  ProxyJump bastion-host-nickname
```

Using the example configuration above, when a an ssh connection is made like so:

```sh
ssh remote-host-nickname
```

...ssh will first create a connection to the bastion host `bastion-hostname` - the host referenced, by nickname, in the `ProxyJump` setting for the remote host, before connecting to the remote host.

## An Alternative: Forwarding stdin and stdout

`ProxyJump` is a simplified way of using a feature ssh has had for a long time already: the `ProxyCommand`.  `ProxyCommand` works by forwarding standard in and standard out from the remote machine though the proxy or bastion hosts. 

The `ProxyCommand` itself is a specific command to be used to connect to a remote server - in this case the manual ssh command that would be used to first connect to the bastion:

```sh
ssh -o ProxyCommand="ssh -W %h:%p bastion-host" remote-host
```

The `%h:%p` arguments to the `-W` flag above specify to forward standard in and out to the remote host (`%h`) and remote host's port (`%p`).

## ProxyCommand in ~/.ssh/config

As with `ProxyJump`, `ProxyCommand` can be set in the `~/.ssh/config` file for hosts that will always use the configuraton. 

```txt
Host remote-host
  ProxyCommand ssh bastion-host -W %h:%p
```

With this set in the `~/.ssh/config` any ssh connection to the remote host will be accomplished by forwarding stdin and stdout through a secure connection from bastion-host.

The ssh command is a powerful tool.  While it might be most used in its simplest form, `ssh user@hostname`, there are literally dozens of uses, flags and configurations to make connections from one host to another.  Check out ssh's manual page (`man ssh`) sometime to discover all the different options available with this seemingly simple program.
