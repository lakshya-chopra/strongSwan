## Installation

### Dependencies
```sh
sudo apt install libcurl4-openssl-dev libssl-dev
sudo apt install libsystemd-dev pkg-config bzip2
```
### StrongSwan
```sh
wget https://download.strongswan.org/strongswan-6.0.0.tar.bz2
tar xjf strongswan-6.0.0.tar.bz2
cd strongswan-6.0.0/
```

#### Configuring StrongSwan
```
./configure --prefix=/usr --sysconfdir=/etc --disable-defaults --enable-silent-rules --enable-charon --enable-systemd \
--enable-ikev2 --enable-vici --enable-swanctl --enable-nonce --enable-random --enable-drbg --enable-openssl --enable-curl \
--enable-pem --enable-x509 --enable-constraints --enable-revocation --enable-pki --enable-pubkey \
--enable-socket-default --enable-kernel-netlink --enable-resolve --enable-eap-identity --enable-eap-md5 \
--enable-eap-dynamic --enable-eap-tls --enable-updown --enable-sha2 \
--enable-pkcs11 --enable-hmac --enable-gcm --enable-hmac --enable-ml

make -j && sudo make install
```

## Service setup
```sh
sudo systemctl enable strongswan.service
sudo systemctl start strongswan.service
```

## Logs
```sh
sudo journalctl -u strongswan --no-pager --since "5 minute ago"
```

Note: if other services of strongSwan are running beside this (for example: starter or etc), then it will lead to errors, for example: [no socket implementation registered](https://github.com/strongswan/strongswan/discussions/2282)

## Configuration

### Core Parameters	
- local_ts:

  `"Comma-separated list of local traffic selectors to include in CHILD_SA. Each selector is a CIDR subnet definition, followed by an optional proto/port selector. The special value dynamic may be used instead of a subnet definition, which gets replaced by the tunnel outer address or the virtual IP if negotiated. This is the default."
  `

- id:

  `"IKE identity to use for authentication round. When using certificate authentication. The IKE identity must be contained in the certificate, either as the subject DN or as a subjectAltName (the identity will default to the certificate’s subject DN if not specified)."
  `

### From the official sources
- ![image](https://github.com/user-attachments/assets/4c802dc0-cdff-4e18-9a3a-69745c803176)

- ![image](https://github.com/user-attachments/assets/22fc49c5-ab01-4388-a9ed-21f7bc173e1a)

- ![image](https://github.com/user-attachments/assets/135f7612-1743-478c-ba11-4cfa167dbf1f)

- cacert: `"The certificates may use a relative path from the swanctl/x509ca directory or an absolute path"` For other certificates, `/etc/swanctl/x509` dir maybe used.
- Private keys should be stored at `/etc/swanctl/private`.
- Post Quantum PSKs - PPKs, can also be used. Add the key in the secrets subsection and set the ```ppk_required=yes```.
  
### Listing certificates:
![image](https://github.com/user-attachments/assets/f9b125bd-b617-45fb-a11f-15ff9e3d0b46)

### 

## Client & Server config
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

6. You may need to do a daemon-reload (at times) so that the configuration changes reflect in strongSwan:
```sh
sudo systemctl daemon-reload
```

Example:

![image](https://github.com/user-attachments/assets/ae362d29-b69d-4a41-9322-6da59ad760b5)


## Advanced Logging
strongSwan supports several levels of logging for IKE, ESP, APP, etc. Here's what the official documentation says: 
| Level | Description                                                                 |
|-------|-----------------------------------------------------------------------------|
|-1     | Absolutely silent                                                           |
| 0     | Very basic auditing logs (e.g., SA up/SA down)                              |
| 1     | Generic control flow with errors, a good default to see what's going on     |
| 2     | More detailed debugging control flow                                        |
| 3     | Including RAW data dumps in hex                                             |
| 4     | Also include sensitive material in dumps, e.g., keys                        |

### Add the levels

```sh
sudo vim /etc/strongswan.conf

# this could also be used -- sudo vim /etc/strongswan.d/charon-logging.conf
```
> Earlier, strongSwan relied on multiple conf dirs including `charon` & `swanctl.conf`. However, in the newer versions, this has been largely replaced by `strongSwan.conf` which serves as the main configuration file, simplifying deployments.


### Edit the charon section to specify logging levels for any of the sources mentioned below

| Tag  | Description                                                     |
|------|-----------------------------------------------------------------|
| app  | Applications other than daemons                                |
| asn  | Low-level encoding/decoding (ASN.1, X.509, etc.)               |
| cfg  | Configuration management and plugins                           |
| chd  | CHILD_SA / IPsec SA                                            |
| dmn  | Main daemon setup/cleanup/signal handling                      |
| enc  | Packet encoding/decoding, encryption/decryption operations     |
| esp  | libipsec library messages                                      |
| ike  | IKE_SA / ISAKMP SA                                             |
| imc  | Integrity Measurement Collector                                |
| imv  | Integrity Measurement Verifier                                 |
| job  | Job queuing/processing and thread pool management              |
| knl  | IPsec / Networking kernel interface                            |
| lib  | libstrongswan library messages                                 |
| mgr  | IKE_SA manager, handles synchronization for IKE_SA access      |
| net  | IKE network communication                                      |
| pts  | Platform Trust Service                                         |
| tls  | libtls library messages                                        |
| tnc  | Trusted Network Connect                                        |


> Note: Add the levels under either filelog or syslog subsections (or both) depending on your requirement. From the official documentation, syslog could be more expensive if it flushes everything to the disk.

As an example,
```md
charon {
    filelog {

        charon{
            path =  /var/log/charon.log
            append = yes # append to the file, instead of overwriting original data

            # Default loglevel.
            default = 4
            ike = 4
            esp = 4
            app = 4
            chd = 4
            job = 4
            net = 4
            tls = 4
            enc = 4
            time_format = %b %e %T # add a formatted Timestamp before the formatted log messages
        }
  }
    # Plugins
    load = aes sha1 sha2 md5 des hmac gmp random nonce x509 pubkey pkcs1 pem

    }
}

```

### Important 
- Never set logging level to 4 unless you're absolutely sure. It may leak sensitive material including keys, secrets, and raw packet data into logs.

- For production, always prefer:
    ```conf
    ike = 1
    esp = 1
    cfg = 1
    ```
- Levels 3/4 are only suitable for debugging in a controlled & isolated environment.

### Verbose logging snippets

For full file, check this: [./log/charon.log](https://github.com/lakshya-chopra/strongSwan/blob/main/log/charon.log)

### Encryption logs
![image](https://github.com/user-attachments/assets/3728a23c-f8c9-4ae4-b690-e5136af2e878)


![image](https://github.com/user-attachments/assets/93db2d17-7bcf-4f4a-bc1a-d487e80eaa1a)


![image](https://github.com/user-attachments/assets/76bc1775-88c3-48e4-b47c-be6b3be067b7)


### IPSec Interface

![image](https://github.com/user-attachments/assets/2985236d-de66-4deb-bfc9-8f63dc6791f0)

### Config & Connection

![image](https://github.com/user-attachments/assets/97e5d9fd-6436-44fb-908d-516dccca5a59)

The above image shows:
  - Use of several Diffie-Hellman Groups: **MODP_2048, MODP_3072, MODP_4096, MODP_8192, MODP_6144**
  - Presence of AEADs such as **AES-GCM, AES-CCM (CTR + CMAC), ChaCha20Poly1305 **
  - Use of legacy ciphers & PRFs.
  - Use of PRF over KDF (slightly pedantic)
  - No **SHA-3**.
  - Use of nistp curves, brainpool curves, Curve25519 & Curve448.




## References

- [Identity Parsing](https://docs.strongswan.org/docs/latest/config/identityParsing.html)
- [Peer config issue](https://github.com/strongswan/strongswan/discussions/799)
- [IKE Auth Code](https://github.com/strongswan/strongswan/blob/master/src/libcharon/sa/ikev2/tasks/ike_auth.c)
