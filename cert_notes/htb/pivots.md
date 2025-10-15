# Pivoting & Port Forwarding

## Dynamic Port Forwarding with SSH & SOCKS

### Local Port forward using SSH
> Most useful for services that are not available to anything but localhost.
* Forward a single port on the local host to a single port on a remote host.
```bash
ssh -L [local port]:localhost:[remote port] [user]@[$IP]
```
* Forward multiple ports
```bash
ssh -L [local port]:localhost:[remote port] -L [local port]:localhost:[remote port] [user]@[$IP]
```

### Dynamic Port Forwarding using SSH (SSH Tunneling over SOCKS)
* Verify proxychains is set up for Dynamic Port Forwarding
```bash
tail -4 /etc/proxychains4.conf

# meanwile
# defaults set to "tor"
socks4  127.0.0.1 9050
```
* Setup the tunnel over SSH
```bash
ssh -D [localport] [user]@[$IP]
```
* Verify
```bash
proxychains4 -q nxc smb [remote subnet] -u '' -p ''
```

