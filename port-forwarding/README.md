
# Port Forwarding 
- Architecture

```bash
client --> 22,80,8080 --> Server 1 --> 22,80,8080,3306 --> Server 2
```

- Rerouting incoming traffic from port 80 to port 8080 using `iptables` and REROUNTING chain

```bash
$ iptables -t nat -A REROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080         # Redirect from 80 to 8080

$ iptables -I INPUT 4 -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT            # Open port 8080 for traffic
```
 
- Rerouting incoming traffic from `port 80` to `port 8080` on the same host 

```bash
$ firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080             # Same host
$ firewall-cmd --add-masqurade          # Relevant for port forwarding
```

- Rerouting incoming traffic from `port 80` on the host to `port 8080` on another host.
- Note the server should have a connection or can connect with the second server port-forwarded to 

```bash
$ firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=<server-ip>
$ firewall-cmd --add-masqurade          # Relevant for port forwarding

```


