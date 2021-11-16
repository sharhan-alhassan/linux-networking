# Linux by doing

## 1. Testing DNS not connecting "www.google.com"
- check the following

```bash
$ cat /etc/hosts                # To see if google.com is assigned to an IP
$ cat /etc/nsswitch.conf        # To verify hosts: files, dnst, ..., myhostname
$ cat /etc/resolve.conf         # Check DNS server entries

$ nmcli con show                # Review network connection. Connection shows "System ens5" or something similar

$ nmcli -f ipv4.dns con show "System ens5"          # Check if DNS IP is assigned 

## ouput -- No dns
ipv4.dns:                               --

$ nmcli dev show                # See components of NIC device

$ nmcli dev show | grep DNS     # Grep out DNS IP (first time, you'll see none)


```


## 2. Monitoring Network Access

- Install `iptraf-ng` and `nc` on both servers (server1 and server2)

- In iptraf configuration on server1, set directory for logging: eg. `/home/cloud_user/traffic_logs.txt`

- Open connection on sever1, port 4040 using `nc`

```bash
$ nc -l 4040                    # Open port for listening
```
- On server2, initiate communication with `nc` or `telnet` or same port

```bash
$ telnet <server1-IP> 4040
```

## 3. Service service log file with "journalctl"


## 4. Configure Network Filesystems for usage

## 5. Sed to change "COWS" to "Ants"

- We're going to run a `sed command`. The `-i` means `"do this in place,"` as in don't create another file. The capital `I` near the end stands for `"case-insensitive"` and means that whether cows has any capital letters in it or not, change it to Ants. The `g` means do it globally, throughout the whole file. Here is the complete command:

`sed -i 's/cows/Ants/Ig' fable.txt`

### Set Up the Samba Server
- Log in to the Samba server using the credentials provided:

```bash
ssh cloud_user@<SAMBA_SERVER_PUBLIC_IP_ADDRESS>

# Become root:
[cloud_user@samba-server]$ sudo -i

# Create the /smb path:
[root@samba-server]# mkdir /smb

# Make sure the client can write to the path:
[root@samba-server]# chmod 777 /smb

# Install the Samba packages:
[root@samba-server]# yum install samba -y

# Open /etc/samba/smb.conf:
[root@samba-server]# vim /etc/samba/smb.conf

# Add the following section at the bottom:


[share]
        browsable = yes
        path = /smb
        writable = yes

# Save and exit the file by pressing Escape followed by :wq.

# Check that our changes saved correctly:

[root@samba-server]# testparm

#Samba Share User
#Create the user on the server:
[root@samba-server]# useradd shareuser

# Give it a password:
[root@samba-server]# smbpasswd -a shareuser

Enter and confirm a password you'll easily remember (e.g., 123456), as we'll need to reenter it later.

# Start the Samba daemon:
[root@samba-server]# systemctl start smb

```