# ssh Tunneling 
- Trying to have access to a second server (which you don't have access to) via a first server (which you have access to)

1. Local ssh tunneling 
Architecture:

ssh --> 8080:server1:80 --> server2
```bash
$ ssh -L 8080:server2:80 cloud_user@server1
```

2. Remote tunneling
- Tunnel port 8080 in server1 back to port 80 on the localhost
```
$ ssh -R 8080:localhost:80 cloud_user@server1
```

3. Dynamic tunneling

```
$ ssh -D 8080 cloud_user@server1
```
