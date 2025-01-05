# bad.example.com Round#1

## Servers

- `bad.example.com` authorized nameserver on `example.com`
  with IP `192.168.57.2/24`

## Network

- Private network `192.168.57.0/24`.
- Requires a Host-only network adapter in your host.

## Deploy

    vagrant up

## Errors found

### named.conf.local

We can see that in that file, the reverse zone path is `/var/lib/bin/192.168.57.dns`, we can see that the `bin` part is wrong
In order to fix it, we need to change `bin` to `bind`.

Also, we can see that in that file, the path for the example zone is `/var/lib/bind/example.com.dns.`.
There is an extra period at the end that we should remove in order for it to work, it should look like this:
`/var/lib/bind/example.com.dns`

### named.conf.options

The recursion is disabled, because of this, it can produce some errors, so its better for us to keep it enabled.
change `recursion no;` to `recursion yes;`

We should also add the line `allow-query { any; };` so everyone is allowed to use queries, this is because
bind9 may disable queries for some users, and the query could fail if that happens.

Also, if we dont need it, `dnssec-validation yes;` could give us problems, so its better to have it shut down,
to do that, we just change it to this: `dnssec-validation no;`

We also have to make sure that we add ; at the end of the options

### example.com.dns

The $ORIGIN is not what it should be, it is set to `$ORIGIN example.org.` when it should be a .com `$ORIGIN example.com.`
Use fully qualified domain names in the zone, `@     IN  NS      bad.` to `@     IN  NS      bad.example.com.`.
For the A register, dont use `bad.  IN  A       192.168.57.2`, we need t o remove the period from bad,
It should look like this: `bad  IN  A       192.168.57.2`.
    
### 192.168.57.dns

The records for the inverse zones are not fully qualified, we need to add a period at the end of them to ensure that they are.
`2   IN	PTR	bad.example.com 3   IN  PTR     www.example.com` to `2   IN	PTR	bad.example.com. 3   IN  PTR     www.example.com.`.
    
### Vagrantfile

We have to be sure that the copied files have the right permissions, so we can add to the vagrant file this lines to ensure that happens.
`chown bind:bind /var/lib/bind/example.com.dns /var/lib/bind/192.168.57.dns`
`chmod 644 /var/lib/bind/example.com.dns /var/lib/bind/192.168.57.dns`

## Test

### Is bind9 active and running?
vagrant@bad:~$ systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
     Active: `active (running)` since Sun 2025-01-05 12:34:14 UTC; 8s ago
       Docs: man:named(8)
   Main PID: 2035 (named)
      Tasks: 8 (limit: 2322)
     Memory: 16.6M
        CPU: 29ms
     CGroup: /system.slice/named.service
             └─2035 /usr/sbin/named -f -u bind

We can see that it is.

### Test nslookup

vagrant@bad:~$ nslookup bad.example.com localhost
Server:         localhost
Address:        ::1#53

Name:   bad.example.com
Address: 192.168.57.2

we can see that it also works

### Ensuring that everything works with debug commands

vagrant@bad:~$ sudo named-checkconf
vagrant@bad:~$ sudo named-checkzone example.com /var/lib/bind/example.com.dns
zone example.com/IN: loaded serial 1
OK

Because `sudo named-checkconf` didn't returned anything, it means that it has no issues, also, because the seccond command 
returned that the zone was loaded, we can confirm that everything went how it should had.