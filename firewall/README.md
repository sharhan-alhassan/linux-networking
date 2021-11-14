## Lab 2: Troubleshooting routes

- Bring up a network interface up (NIC)
```
$ nmcli con up <conn-name>      //eg: Wired\ Connection\ 1
```
- You can set your Routing tables entry to go into specific NICs
- Block your host from accessing `www.google.com`
```
$ ip route add prohibit <google-ip>
```


# Lab 2: Troubleshooting Firewalls using IPTables and Firewalld

## IPTables
- `Netfilter`: Is a framework provided by the Linux kernel for packet filtering. Permit Network address translation, port translation
- Is the engine that runs all firewal
- `IPTables`: The mechanism that allows the implementation of firewall

`IP Tables`:
- `Filter Table`: For filtering packets(allow or deny packet to continue)
- `NAT Table`: For modifying packet source and destination using IP changes to effect routing
- `Mangle Table`: For altering the IP headers of a packet in order to modity the TTL, hops, etc
- `Raw Table`: For opting out of connection tracking
- `Security Table`: For setting `SELinux` security context values on packets or connections

`Examples of IPTables Rule`:
1. Accept all connections on port 22, using filter table, add to the INPUT chain, tcp port
```bash
$ iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT
```

2. Reject all connections from this specified subnet. Insert to `INPUT` chain at line `5`, with a source(s) `10.0.0.1/24`, and jump `j` to `REJECT`
```bash
$ iptables -I INPUT -s 10.0.0.1/24 -j REJECT
```

`Connection Tracking`: You can check you existing connection tracking information at:

```bash
$ cat /proc/net/nf_conntrack
```
### Note: 
- In IPTables, order matters. `REJECT` rule coming before `ACCEPT` rule will make the `ACCEPT` rule blocked
### Firewalld
- `Firewalld` is a frontend controller or wrapper for Iptables
- It's concept of `zones` makes it apart from `iptables`
- Each NIC of host has a default `zone`
- `Zones` are predefined rules that specify what traffic should be allowed based on trust levels for network connections
- check zones in ubuntu at `/usr/lib/firewalld/zones` exists as XML files

# Troubleshooting Firewall: IPTables
Problem: Trying to have access to port 80 (that's why we use curl)

`Architecture`: Client(10.0.1.10) ---> Firewalld ---> host(10.0.1.11)

- Verify service is running with `ss -lntp`     //For port 80
- Verify you can curl it or not: `curl -I localhost`
- Try curling the host header from client. 

```bash
[user@10.0.1.10]$ curl -I 10.0.1.11
```
output:

`curl: (7) Failed connect to 10.0.1.10:80; No route to host`

- Take a look at the `iptables`
- Check order of `ACCEPT` and `REJECT` for the service
- Move `ACCEPT` for `80/tcp` up above any `REJECT` rule in `/etc/sysconfig/iptables`

```bash
$ iptables -vnL               # v(verbose),n(port numb),L(list)
```

# Troubleshooting Firewall: firewalld
- Get your running config:
`$ firewall-cmd --list-all`

- Now let's break it by using `rich-rule` to reject our source(client) to have accept to port 80 on our host

```
$ firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address 10.0.1.10/24 port port=80 protocol=tcp reject
```

- Reload firewall
`$ firewall-cmd --reload`

- Remove the rule with `--remove-rich-rule`
```
$ firewall-cmd --permanent --remove-rich-rule='rule family=ipv4 source address 10.0.1.10/24 port port=80 protocol=tcp reject
```

- Order too matters in Firewalld. Placing a Rejected rule with IP with its subnet(/<subnet>) blocks all other/ takes prescendence of IPs that even have accepted rules below it

# Troubleshooting firewall with new Zone
- Very the service (http) is running

`$ ss -lntp | grep :80`

- Check Firewalld is up and running

`$ systemclt status firewalld`

- Check to see what is accepted and blocked

`$ firewall-cmd --list-all`

- Output

```bash
rule family="ipv4" source address="10.0.1.0/24" port port="80" protocol="tcp" reject
```

- Let's create new zone

`$ firewall-cmd --permanent --new-zone=api`

- Reload
`$ firewall-cmd --reload`

- Add `http` service to it

`$ firewall-cmd --permament --zone-api --add-service=http`

- Add source of IP to the zone

`$ firewall-cmd --permanent --zone=api --add-source=<client-IP>`

- Reload firewalld

`$ firewall-cmd --reload`

- Log into the source(client) and curl the host again

`$ curl -I <destination-IP>`


# Linux Boot process
```bash

$ dig <domain-name> +nssearch               # Check synchronization
```
- Verification

