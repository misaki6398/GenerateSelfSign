# Generate the self signed cer including pfx and jks

## Gen root ca
### Gen private key
```shell
openssl genrsa -aes256 -out RootCa.key 2048
```

### Gen public key
```shell
openssl req -new -x509 -days 10950 -sha256 -config ./RootCa.cnf -key RootCa.key -out RootCa.crt -subj "/CN=ROOT-CA/OU=MYSELF/O=GOGOPF/L=TAIPEI/ST=TAIWAN/C=TW"
```

### Cat pem file if needed
```shell
cat MegaRootCa.crt MegaRootCa.key > MegaRootCa.pem
```

## Gen sub ca
### Gen private key
```shell
openssl genrsa -aes256 -out SubCa.key 2048
```

### Gen csr
```shell
openssl req -new -key SubCa.key -out SubCa.csr -subj "/CN=MEGA-SUB-CA/OU=AP11/O=COMPANY/ST=TAIWAN/C=TW"
```

### Gen public key
```shell
openssl x509 -req -days 10950 -sha256 -extfile SubCa.cnf -CAcreateserial -CA RootCa.crt -CAkey RootCa.key -in SubCa.csr -out SubCa.crt
```

### Cat pem file if needed
```shell
cat SubCa.crt SubCa.key > SubCa.pem
```

## Gen Server key
```shell
openssl genrsa -aes256 -out Server.key 2048
```

### Gen csr
```shell
openssl req -new -key Server.key -out Server.csr -subj "/CN=*.yourdomain.com.tw/O=COMPANY/L=TAIPEI/ST=TAIWAN/C=TW"
```

### Gen public key
```shell
openssl x509 -req -days 10950 -sha256 -extfile Server.cnf -CAcreateserial -CA SubCa.crt -CAkey SubCa.key -in Server.csr -out Server.crt

```

### Cat pem file if needed
```shell
cat Server.crt Server.key > Server.pem
```

```shell
cat Server.pem SubCa.pem RootCa.pem > ALL.pem
```

### Gen pfx
```shell
openssl pkcs12 -export -in ALL.pem -out Server.pfx -name aliasname
```

## Import key and trustcacert to jks
```shell
keytool -importkeystore -srckeystore "Server.pfx" -srcstoretype JKS -destkeystore "Server.jks" -deststoretype PKCS12 
keytool -keypasswd -alias tfedq -keystore Server.jks 
keytool -import -trustcacerts -file SubCa.pem -alias subca -keystore Server.jks 
keytool -import -trustcacerts -file RootCa.pem -alias rootca -keystore Server.jks  
```