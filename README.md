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
