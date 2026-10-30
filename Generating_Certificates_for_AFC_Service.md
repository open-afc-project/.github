# Generating Certificates for AFC Server:


These steps can be done in a seprate folders, with the outputs moved the required locations.
## Generate CA running:
```
openssl req -new -x509 -days 9999 -keyout test_ca_key.pem -out test_ca_crt.pem
```


## Generate Server SSL Certificate, Key:
```
openssl genrsa -out server.key.pem 4096
```

```
openssl req -new -key server.key.pem -out server.csr.pem
```


## Sign with proper DNS and IP:

Relvant Information: Extfile upgrades the version to v3 which is now required.

```
openssl x509 -req \
    -in server.csr.pem \
    -CA test_ca_crt.pem \
    -CAkey test_ca_key.pem \
    -CAcreateserial \
    -out server.cert.pem \
    -days 9999 \
    -sha256 \
    -extfile <(printf "subjectAltName=DNS:localhost,DNS:server.localhost,IP:127.0.0.1\\nbasicConstraints=CA:FALSE\\nkeyUsage=digitalSignature,keyEncipherment\\nextendedKeyUsage=serverAuth")
```

```
cat server.cert.pem test_ca_crt.pem > server.bundle.pem
```


## Generate Client SSL Certificate, Key:


```
openssl genrsa -out client.key.pem 4096
```

```
openssl req -new -key client.key.pem -out client.csr.pem
```

```
openssl x509 -req -days 9999 -in client.csr.pem -CA ca.cert.pem -CAkey ca.key.pem -CAcreateserial -out client.cert.pem
```

```
openssl verify -CAfile test_ca_crt.pem client.cert.pem
```

```
cat test_ca_crt.pem client.cert.pem > client.bundle.pem
```




## After Creation:


Inside openafc/dispatcher/servers:

- Replace the server files with server.key.pem, server.cert.pem, server.bundle.pem

- Replace the client files with test_ca_crt.key, test_ca_key.pem, client.bundle.pem

The others should be saved, with clients being able to be added through mTLS on server as needed.

