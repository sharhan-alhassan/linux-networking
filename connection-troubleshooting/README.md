# Connection troubleshooting

```
$ nmcli --help          # cdm  grancdm
OBJECT
  g[eneral]       NetworkManager's general status and operations
  n[etworking]    overall networking control
  r[adio]         NetworkManager radio switches
  c[onnection]    NetworkManager's connections
  d[evice]        devices managed by NetworkManager
  a[gent]         NetworkManager secret agent or polkit agent
  m[onitor]       monitor NetworkManager changes

```

# OSI Layers
- Layer 2: DataLink(MAC) -- switch and VLAN configuration, MAC addressing and IP conflicts
Tools: arping

- Layer 3: Network(IP) -- addressing and routing, bandwidth, network authentication
Tool: ping, tracepath, traceroute,

- Layer 4: Transport -- blocked ports, firewalls
Tools: ss, telnet, tcpdump, netcat(nc),

- Layer 5-7: Application/service function
Tools: dig, service tools

# Troubleshooting
A broken server with 2 NICs
1. eth0: 10.0.0.1.10
2. eth1: 10.0.1.20  with port=80 for http traffic
client: 10.0.1.11 

- First Resolve DNS issue if requied. Check `dns-resolution` doc

1. Verify issues exist with:

```bash
$ curl -I 10.0.1.20                 # NIC with port=80 on server
```

2. Verify the service is actually running (httpd) on server

```bash
$ ss -lntp | grep :80
```

3. If is not running, start the service(httpd)

```bash
$ sudo systemctl start httpd
```

4. Check the firewalld if the service is in there
```bash
$ firewall-cmd --list-all
```

5. If `http` is not in firewalld, add it

```bash
$ firewall-cmd --permanent --add-service=http
```

6. Reload firewall

```bash
$ firewall-cmd --reload
```

7. Use `nc` to listen to port 80 on the server and `telnet` on the client to initiate communication with the server

```bash
$ sudo nc -l -p 80                      # On the server
$ telent 10.0.1.20 80
```

8. Now proceed to Layer 3 (Network) -- routing information(route table).
This is to resolve `asymmetric routing`

```bash
$ route -n

# Output
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.43.105  0.0.0.0         UG    600    0        0 wlo1
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 wlo1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.43.0    0.0.0.0         255.255.255.0   U     600    0        0 wlo1

```

9. Add the destination IP (with subnet) to the devices(eth1) route table. Add rule 

```bash
$ ip route add 10.0.1.0/24 dev eth1 tab 1
$ ip rule add from 10.0.1.0/24 tab 1
```

### Recap
- verify the service - layer 5
- verify firewall - layer 4
- verify routing information (route table) - Layer 3



telnet and netcat very important
tcpdump -- captures the packets or data
