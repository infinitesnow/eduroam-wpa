# `wpa_supplicant` configuration for eduroam and polimi-protected

For some people (like me) the official eduroam installer won't work. I explain here how to manually configure `wpa_supplicant`.

First, the configuration for `wpa_supplicant` which has to be appended in `/etc/wpa_supplicant.conf`:

```
network={
        ssid="eduroam"
        key_mgmt=WPA-EAP
        pairwise=CCMP
        group=CCMP TKIP
        eap=TLS
        ca_cert="/path/to/ca.pem"
	client_cert="/path/to/user.pem"
	private_key="/path/to/key.pem"
        identity="XXXXXXXX@polimi.it"
        altsubject_match="DNS:wifi.polimi.it"
        phase2="auth=MSCHAPV2"
}

network={
        ssid="polimi-protected"
        key_mgmt=WPA-EAP
        pairwise=CCMP
        group=CCMP TKIP
        eap=TLS
        ca_cert="/path/to/ca.pem"
	client_cert="/path/to/user.pem"
	private_key="/path/to/key.pem"
        identity="XXXXXXXX@polimi.it"
        altsubject_match="DNS:wifi.polimi.it"
        phase2="auth=MSCHAPV2"
}
```

`XXXXXXXX` is your 8-digit Person Code (10XXXXXX).

`ca.pem` is in `/etc/ssl/certs/DigiCert_Assured_ID_Root_CA.pem` on Debian, otherwise it can be extracted from the  official `eduroam-linux-PdM-polimi-TLS.py` installer: 

```perl -e '$f=join("",<>); print $& if $f=~/-+BEGIN CERTIFICATE-+(\n.*)*-+END CERTIFICATE-+/m' eduroam-linux-PdM-polimi-TLS.py > /path/to/ca.pem```

If this does not work, just copy the text 

```
-----BEGIN CERTIFICATE-----
...VERY_BIG_RSA_PRIME_NUMBERS...
-----END CERTIFICATE-----
```

by hand from the Python script and save it to `/path/to/ca.pem`

`wpa-supplicant` does not work with PKCS12 certificates, thus they must be unpacked to `.pem`.

`openssl pkcs12 -in wifiPolimi.p12 -out /path/to/user.pem  -nokeys` will extract the certificate (`-nokeys`) 

`openssl pkcs12 -in wifiPolimi.p12 -out /path/to/key.pem -nocerts -nodes` will extract the private key (`-nocerts`) without encrypting it (`-nodes`, no-Data-Encryption-Standard).

If one wishes to keep the private key password protected, skip the `-nodes` flag and put this line in the each network configuration:

```private_key_passwd="password"```

**Note for dummies: obviously you choose what ```/path/to/``` values is.**
