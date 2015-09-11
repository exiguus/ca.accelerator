SET OF BASH SCRIPTS FOR EASY HTTPS CERT MANAGEMENT
creating certificate requests and (pound like) pem files
pound is a HTTPS front-end for Web server

SETUP
chmod +x make-*
vi openssl.cnf


RUN
./make-csr
make a certificate request in /csr and a private key in /key

./make-pound-crt
make a pound ready retrieved certificate from existing certificate in /crt

./make-pound-key
make a pound ready private key from exiting key in /key

./make-pound-pem
make a complete pem file for pound with startssl cert for example in /pem
(make-pound-crt and make-pound-key run within make-pound-key)

./make-private-ca
make a private ca with passphrase

OTHER FILES
HOWTO.txt describe howto setup encrypted connections for webserver, mail and ftp with configuration example
openssl.cnf to configure your default cert settings
README.txt see this file
startssl-ca.crt is the intermediate cert from startssl
