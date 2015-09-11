#HOWTO configure encrypted service connections for your website, ftp and email server

##TOC
1. https
1.1. private and public key
1.2. cert-request
1.3 pound pem
1.4 pound https
2. CA file
3. email
3.1 postifx smtps
3.2 dovecot imaps/pops
4. ftp
4.1 proftp ftps



## 1. https
#### 1.1 private and public key
		Simon@nike:~ $ [130] openssl genrsa -out example.com.key 4096

### 1.2 public key for cert-request
		Simon@nike:~ $ [130] openssl req -new -key secure.example.org.key -out secure.example.org.csr
		You are about to be asked to enter information that will be incorporated
		into your certificate request.
		What you are about to enter is what is called a Distinguished Name or a DN.
		There are quite a few fields but you can leave some blank
		For some fields there will be a default value,
		If you enter '.', the field will be left blank.
		-----
		Country Name (2 letter code) [AU]:DE
		State or Province Name (full name) [Some-State]:
		Locality Name (eg, city) []:MyCity
		Organization Name (eg, company) [Internet Widgits Pty Ltd]:MyCompany
		Organizational Unit Name (eg, section) []:
		Common Name (e.g. server FQDN or YOUR name) []:secure.example.org
		Email Address []:hostmaster@example.org

		Please enter the following 'extra' attributes
		to be sent with your certificate request
		A challenge password []:
		An optional company name []:


### 1.3 pound pem
Pound 1-File PEM
		-----BEGIN RSA PRIVATE KEY-----

		...

		...  Private RSA key

		...

		-----END RSA PRIVATE KEY-----

		-----BEGIN CERTIFICATE-----

		...

		...  Public key cert

		...

		-----END CERTIFICATE-----

		-----BEGIN CERTIFICATE-----

		...

		...  Intermediate issuers cert

		...

		-----END CERTIFICATE-----

		-----BEGIN CERTIFICATE-----

		...

		...  Root CA cert

		...

		-----END CERTIFICATE-----

### 1.4 pound https
		ListenHTTPS
			Address 0.0.0.0
			Port	443

			Cert "/etc/ssl/startssl/pound/secure.example.org.pem"
			Cert "/etc/ssl/startssl/pound/example.net.pem"
		  ...

## 2. CA file
generate CA-file from startssl for example to use in postfix, dovecot and proftp
		wget --no-check-certificate https://www.startssl.com/certs/ca-bundle.pem -O startssl-ca-bundle.pem
		cat startssl-ca-bundle.pem >> ca-bundle.crt

## 3. mail
## 3.1 postfix smpts
		# main.cf
		### TLS settings
		###
		## TLS for outgoing mails from the server to another server
		smtp_tls_security_level = may
		smtp_tls_note_starttls_offer = yes
		## TLS for email client
		smtpd_tls_security_level = may
		smtpd_tls_cert_file = /etc/ssl/startssl/mail/secure.example.org.pem
		smtpd_tls_key_file = /etc/ssl/startssl/secure.example.org.key
		#$smtpd_tls_cert_file
		smtpd_tls_CAfile = /etc/ssl/startssl/ca-bundle.crt
		smtpd_tls_loglevel = 1
		smtpd_tls_received_header = yes

		# new server side
		smtpd_use_tls = yes
		smtpd_tls_auth_only = yes
		smtpd_tls_received_header = yes

		smtpd_tls_session_cache_timeout = 3600s
		#smtpd_tls_session_cache_database = btree:/var/spool/postfix/smtpd_tls_cache
		#smtpd_tls_loglevel = 3

		smtpd_tls_mandatory_protocols = SSLv3,TLSv1
		smtpd_tls_mandatory_ciphers = medium
		smtpd_tls_cipherlist = HIGH:MEDIUM:+TLSv1:!SSLv2:+SSLv3

		tls_random_source = dev:/dev/urandom

		# new client side
		smtp_tls_mandatory_ciphers = medium
		smtp_tls_mandatory_exclude_ciphers = RC4, MD5
		smtp_tls_exclude_ciphers = aNULL
		smtp_tls_cipherlist = HIGH:MEDIUM:+TLSv1:!SSLv2:+SSLv3

		# 3.2 dovecot imaps/pops
		ssl_cipher_list = HIGH:MEDIUM:+TLSv1:!SSLv2:+SSLv3
		ssl = required # v1.2.beta1+ uses ssl = yes
		# Preferred permissions: root:root 0444
		ssl_cert_file = /etc/ssl/startssl/mail/secure.example.org.pem
		# Preferred permissions: root:root 0400
		ssl_key_file = /etc/ssl/startssl/secure.example.org.key
		# ca
		ssl_ca_file = /etc/ssl/startssl/ca-bundle.crt

## 4. ftp
### 4.1 proftp ftps
		TLSRSACertificateFile                   /etc/ssl/startssl/secure.example.org.crt
		TLSRSACertificateKeyFile                /etc/ssl/startssl/secure.example.org.key
		#
		# CA the server trusts
		TLSCACertificateFile 			 		/etc/ssl/startssl/ca-bundle.crt
