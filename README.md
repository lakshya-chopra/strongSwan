## Installation

Dependencies:
```sh
sudo apt install libcurl4-openssl-dev
sudo apt install libsystemd-dev
```
StrongSwan:
```sh
wget https://download.strongswan.org/strongswan-6.0.0.tar.bz2
tar xjf strongswan-6.0.0.tar.bz2
cd strongswan-6.0.0/

./configure --prefix=/usr --sysconfdir=/etc --disable-defaults --enable-silent-rules --enable-charon --enable-systemd \
--enable-ikev2 --enable-vici --enable-swanctl --enable-nonce --enable-random --enable-drbg --enable-openssl --enable-curl \
--enable-pem --enable-x509 --enable-constraints --enable-revocation --enable-pki --enable-pubkey \
--enable-socket-default --enable-kernel-netlink --enable-resolve --enable-eap-identity --enable-eap-md5 \
--enable-eap-dynamic --enable-eap-tls --enable-updown --enable-sha2 \
--enable-pkcs11 --enable-hmac --enable-gcm --enable-hmac --enable-ml

make -j && sudo make install
```
Start the service:
```sh
sudo systemctl enable strongswan.service
sudo systemctl start strongswan.service
```
View logs:
```sh
sudo journalctl -u strongswan --no-pager --since "5 minute ago"
```

Note: if other services of strongSwan are running beside this (for example: starter or etc), then it will lead to errors, for example: [no socket implementation registered](https://github.com/strongswan/strongswan/discussions/2282

## Configuration:

### Certain parameters:
- local_ts:

  `"Comma-separated list of local traffic selectors to include in CHILD_SA. Each selector is a CIDR subnet definition, followed by an optional proto/port selector. The special value dynamic may be used instead of a subnet definition, which gets replaced by the tunnel outer address or the virtual IP if negotiated. This is the default."
  `

- id:

  `"IKE identity to use for authentication round. When using certificate authentication. The IKE identity must be contained in the certificate, either as the subject DN or as a subjectAltName (the identity will default to the certificate’s subject DN if not specified)."
  `

- ![image](https://github.com/user-attachments/assets/4c802dc0-cdff-4e18-9a3a-69745c803176)

- ![image](https://github.com/user-attachments/assets/22fc49c5-ab01-4388-a9ed-21f7bc173e1a)

- ![image](https://github.com/user-attachments/assets/135f7612-1743-478c-ba11-4cfa167dbf1f)

- cacert: `"The certificates may use a relative path from the swanctl/x509ca directory or an absolute path"` For other certificates, `swanctl/x509` dir maybe used.
![image](https://github.com/user-attachments/assets/f9b125bd-b617-45fb-a11f-15ff9e3d0b46)

### 

## Client & Server config:
1. For testing purposes, these are the example client & server configs you can use:
[client](https://github.com/lakshya-chopra/strongSwan/blob/main/client/swanctl.conf)
[server](https://github.com/lakshya-chopra/strongSwan/blob/main/server/swanctl.conf)

2. Load the config files & certs:
```sh
swanctl --load-all
swanctl --load-creds
```

3. Initiate IKE & Child IKE SA:
```sh
swanctl --initiate --ike conn
swanctl --initiate --child child
```

4. View the logs via `journalctl`, or by configuring strongSwan to use a logging file.

5. List the currently active SAs & conns:
```sh
swanctl --list-sas
swanctl --list-conns
```

Example:

![image](https://github.com/user-attachments/assets/ae362d29-b69d-4a41-9322-6da59ad760b5)



## References:

- [Identity Parsing](https://docs.strongswan.org/docs/latest/config/identityParsing.html)

- [Peer config issue](https://github.com/strongswan/strongswan/discussions/799)
