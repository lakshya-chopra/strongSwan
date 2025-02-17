# strongSwan

## Installation:
```sh
sudo apt update && sudo apt install -y strongswan strongswan-pki
```

## Generate Private CA & Certificates:
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
sudo cp server.key /etc/ipsec.d/private/
```
