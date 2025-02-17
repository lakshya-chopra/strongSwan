# strongSwan

## Installation:
```sh
sudo apt update
sudo apt install -y strongswan strongswan-pki libcharon-extra-plugins libcharon-extauth-plugins \
libstrongswan-extra-plugins libtss2-tcti-tabrmd0 
```
Other installation methods include building strongSwan from source where you can add `./config` options.
[View here](https://docs.strongswan.org/docs/5.9/install/autoconf.html)

## Enable the service:
```sh
systemctl status strongswan-starter
systemctl status strongswan-starter
```

## strongSwan VPN tunnel between 2 peers:

## Setup IP forwarding:
```sh
sysctl net.ipv4.ip_forward=1
sysctl net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.accept_redirects = 1
```
Append this to `/etc/sysctl.conf` and run `sudo sysctl -p`

### Generate Private CA & Certificates:
1. CA:
```sh
 openssl genpkey -algorithm RSA -out ca.key
 openssl req -key ca.key -new -x509 -out ca.crt -days 3650 -subj "/CN=Private CA"
```
2. Use the CA cert to issue certificates to the peers:
```sh
 openssl genpkey -algorithm RSA -out server.key
 openssl req -new -key server.key -out server.csr -subj "/CN=192.168.6.233"
 openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365
```
Do the same for client, and copy the certificates to the strongSwan `sysconfdir`:
```sh
sudo cp ca.crt /etc/ipsec.d/cacerts/
sudo cp server.crt /etc/ipsec.d/certs/
sudo cp client.crt /etc/ipsec.d/certs/
sudo cp server.key /etc/ipsec.d/private/
```
`ipsec-pki` can also be used for certificate & CA generation.

[Note: The certificates are acting as some sort of PSK, and so must be shared between the two peers via a secure & trusted channel.]

## Legacy (swanctl.conf is now recommended)
3. Edit `/etc/ipsec.conf`:
Server:
```conf
config setup
    charondebug="all"
    uniqueids=no

conn %default
    keyexchange=ikev2
    ikelifetime=60m
    keylife=20m
    rekeymargin=3m
    keyingtries=1        # retries for SA establishment

conn myvpn
    type=tunnel
    left=192.168.6.233          #local IP
    leftcert=server.crt         # own certificate
    leftsubnet=10.10.10.0/24    # Local subnet
    #leftid=<peer-cert-subject>
    leftfirewall=yes
    leftauth=pubkey
    right=192.168.6.232         # Peer IP addr
    rightsubnet=10.10.10.0/24
    rightid=@client
    rightcert=client.crt
    rightauth=pubkey
   # rightid=<peer-cert-subject>
    authby=pubkey
    aggressive=no
    auto=start                  
```

Client:
```conf
config setup
    charondebug="all"
    uniqueids=no

conn %default
    keyexchange=ikev2
    ikelifetime=60m
    keylife=20m
    rekeymargin=3m
    keyingtries=1        # retries for SA establishment

conn myvpn
    type=tunnel
    left=192.168.6.232          #local IP
    leftcert=client.crt         # own certificate
    leftsubnet=10.10.10.0/24    # Local subnet
    leftfirewall=yes
    #leftid=@client
    leftauth=pubkey
    right=192.168.6.233         # Peer IP addr
    rightsubnet=10.10.10.0/24
    rightcert=server.crt
    #rightid=@server
    rightauth=pubkey
    authby=pubkey
    aggressive=no
    auto=start                  

```

More: [Conf options](https://wiki.strongswan.org/projects/strongswan/wiki/IpsecConf)

3. Start the ipsec server & client:
```sh
sudo ipsec restart
```
```sh
sudo ipsec up myvpn
```
![image](https://github.com/user-attachments/assets/ddbc5e5e-dd6d-4e8d-a50b-42f0603ee350)


4. Check status:
```sh
sudo ipsec statusall
```
![image](https://github.com/user-attachments/assets/87bd515b-4b82-4f8f-8587-cd1f342c7c9e)

5. Using ip xfrm:
```sh
sudo ip xfrm state
```
