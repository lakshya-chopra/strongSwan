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
./configure --prefix=/usr --sysconfdir=/etc --disable-defaults --enable-silent-rules      --enable-charon --enable-systemd --enable-ikev2 --enable-vici --enable-swanctl        --enable-nonce --enable-random --enable-drbg --enable-openssl --enable-curl           --enable-pem --enable-x509 --enable-constraints --enable-revocation --enable-pki      --enable-pubkey --enable-socket-default --enable-kernel-netlink --enable-resolve      --enable-eap-identity --enable-eap-md5 --enable-eap-dynamic --enable-eap-tls          --enable-updown --enable-sha2 --enable-pkcs11 --enable-hmac --enable-gcm --enable-hmac
make -j && sudo make install
```
Start the service:
```sh
sudo systemctl enable strongswan.service
sudo apt install libcurl4-openssl-dev
```

## Configuration:

local_ts:
```md
"Comma-separated list of local traffic selectors to include in CHILD_SA. Each selector is a CIDR subnet definition, followed by an optional proto/port selector. The special value dynamic may be used instead of a subnet definition, which gets replaced by the tunnel outer address or the virtual IP if negotiated. This is the default."
```
id:
```md
IKE identity to use for authentication round. When using certificate authentication. The IKE identity must be contained in the certificate, either as the subject DN or as a subjectAltName (the identity will default to the certificate’s subject DN if not specified). 
```

![image](https://github.com/user-attachments/assets/4c802dc0-cdff-4e18-9a3a-69745c803176)

![image](https://github.com/user-attachments/assets/22fc49c5-ab01-4388-a9ed-21f7bc173e1a)

![image](https://github.com/user-attachments/assets/135f7612-1743-478c-ba11-4cfa167dbf1f)


## References:
[Identity Parsing](https://docs.strongswan.org/docs/latest/config/identityParsing.html)
