# Generating Certificates for AFC Server:



##### Generate CA running:

openssl req -new -x509 -days 9999 -keyout test\_ca\_key.pem -out test\_ca\_crt.pem



##### Generate Server SSL Certificate, Key:

openssl genrsa -out server.key.pem 4096



openssl req -new -key server.key.pem -out server.csr.pem



##### Sign with proper DNS and IP:

\#Note extfile upgrades the version to v3 which is now required.

openssl x509 -req \\

&#x20; -in server.csr.pem \\

&#x20; -CA test\_ca\_crt.pem \\

&#x20; -CAkey test\_ca\_key.pem \\

&#x20; -CAcreateserial \\

&#x20; -out server.cert.pem \\

&#x20; -days 9999 \\

&#x20; -sha256 \\

&#x20; -extfile <(printf "subjectAltName=DNS:localhost,DNS:server.localhost,IP:127.0.0.1\\nbasicConstraints=CA:FALSE\\nkeyUsage=digitalSignature,keyEncipherment\\nextendedKeyUsage=serverAuth")



cat server.cert.pem test\_ca\_crt.pem > server.bundle.pem



##### Generate Client SSL Certificate, Key:



openssl genrsa -out client.key.pem 4096



openssl req -new -key client.key.pem -out client.csr.pem



openssl x509 -req -days 9999 -in client.csr.pem -CA ca.cert.pem -CAkey ca.key.pem -CAcreateserial -out client.cert.pem



openssl verify -CAfile test\_ca\_crt.pem client.cert.pem



cat test\_ca\_crt.pem client.cert.pem > client.bundle.pem





##### After Creation:



Inside openafc/dispacter/servers:

Replace the server files with server.key.pem, server.cert.pem, server.bundle.pem

Replace the client files with test\_ca\_crt.key, test\_ca\_key.pem, client.bundle.pem



The others should be saved, with clients being able to be added through mTLS on server as needed.

