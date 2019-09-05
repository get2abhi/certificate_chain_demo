# certificate_chain_demo
Basic example of chaining certificates


### Create Root CA

##### Create Root Key
``` sh
openssl genrsa -aes256 -out ROOT_CA.key 2048
```

##### Create Root Cert with this Root Key
```
openssl req -key ROOT_CA.key -new -x509 -days 7300 -sha256 -out ROOT_CA.pem
```
