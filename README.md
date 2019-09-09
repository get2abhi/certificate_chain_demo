# Certificate Chaining PoC for Root, Intermediate and end device

## Create ROOT CA

##### Generate Private key
```sh
openssl genrsa -aes256 -out ROOT_CA.key 2048
```

##### Generate CSR with above key
```sh
openssl req -key ROOT_CA.key -new -x509 -days 7300 -sha256 -out ROOT_CA.pem
```

## Create Intermediate CA Key

```sh
openssl genrsa -aes256 -out Intermediate.key.pem 2048
```

##### Generate CSR with above key
```sh
openssl req -addext basicConstraints=critical,CA:TRUE -new -sha256 -key Intermediate.key.pem -out Intermediate.csr.pem
```

## Verify Intermediate's CSR
```sh
openssl req -verify -in Intermediate.csr.pem -text -noout
```

## Sign this Intermediate CSR by Root CA
```sh
mkdir demoCA
touch demoCA/index.txt
echo 1122334455667788 > demoCA/serial
openssl ca -policy policy_anything -extensions v3_ca -days 3650 -outdir . -cert ROOT_CA.pem -keyfile ROOT_CA.key -in Intermediate.csr.pem -out Intermediate.crt
```

## Dump certificate information
```sh
openssl x509 -in Intermediate.crt -noout -text
```

## Verify if root CA signed Intermediate
```sh
openssl verify -CAfile ROOT_CA.pem Intermediate.crt
```

## Create DEVICE_XXX certificate key
```sh
openssl genrsa -aes256 -out DEVICE_XXX.key.pem 2048
```

## For Elliptical Curve Key use. 
This is ideal for IoT devices because it gives the security of 2048 in 256 bit of randomness with low compute power
```sh
openssl ecparam -name secp384r1 -genkey -noout -out secp384r1.pem
```

## Generate DEVICE_XXX CSR
```sh
openssl req -new -sha256 -key DEVICE_XXX.key.pem -out DEVICE_XXX.csr.pem
```
## Sign CSR by Intermediate Key
```sh
openssl ca -days 3650 -outdir . -cert Intermediate.crt -keyfile Intermediate.key.pem -in DEVICE_XXX.csr.pem -out DEVICE_XXX.crt
```

## Verify the complete chain
```sh
openssl verify -CAfile ROOT_CA.pem -untrusted Intermediate.crt DEVICE_XXX.crt
```

## Convert certificate to pem format
```sh
openssl x509 -outform pem -in DEVICE_XXX.crt -out DEVICE_XXX.pem
```
