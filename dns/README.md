## DNS
```bash
$ hosts <domain-name>               # Get IP of domain name
$ getent ahosts <domain-name>       # Get IP and stream type (DGRAM, etc)
$ curl -I <domain-name>             # Get the header info
$ dig <domain-name>                 # DNS lookup
```
- DNS Lookup sequence
- cat /etc/nsswitch.conf: 
Output:
```
hosts:          files mdns4_minimal [NOTFOUND=return] dns
```
- Means, `Domain name` is first tried to be resolves in the files (in `etc/hosts`), if not found then it goes out to the `dns` server

## Setting DNS hosts and Name servers
```
$ dig NS <domain-name>
```
output
```
; <<>> DiG 9.16.8-Ubuntu <<>> NS cloudbintech.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1169
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;cloudbintech.com.		IN	NS

;; ANSWER SECTION:
cloudbintech.com.	1800	IN	NS	ns3.digitalocean.com.
cloudbintech.com.	1800	IN	NS	ns1.digitalocean.com.
cloudbintech.com.	1800	IN	NS	ns2.digitalocean.com.
```

# Troubleshooting DNS Issues
Problem: Client complains he can't reach `cloudbintech.com`

Problem Architecture: client(10.0.1.10) --> DNS --> Taget (Cloudbintech)

Solution:
1. Curl the target header: `curl -I cloudbintech.com`. If it can't retrieve the header
2. Check `/etc/nsswitch.conf`. Make sure to see this in the hosts:
```
hosts: files mdns4_minimal [NOTFOUND=return] dns
```
- Add `dns` in the hosts if not found.
3. Check `/etc/hosts/` to make sure `cloubintech` is not matched with the loopback(local) IP or any static IP
```bash
127.0.0.1   cloudbintech.com
```
If found remove it

4. Curl the header again: `curl -I cloudbintech.com`. If not connecting...continue to 5

5. Fetch the `nameserver IP` from `/etc/resolve.conf` and try connecting the port 53 (route53)
```bash
$ telnet <nameserver-ip> 53             # IP: 10.0.1.1
```
6. If it's not connecting, there is a possible `DNS IP 10.0.0.2` kicking in. Test with that
```bash
$ telnet 10.0.0.2 53                      # Connect to port 53
$ dign @10.0.0.2 cloudbintech.com         # Verify that
```
7. Restart connection to pick up the new DNS IP
```bash
$ nmcli con mod System\ eth0 ipv4.dns 10.0.0.2
$ systemctl restart network
$ cat /etc/resolv.conf                    # Verify to check it picked up 10.0.0.2 as new nameserver
```

