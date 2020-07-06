---
date: Wed, 03 Feb 2019 00:54:56 -0400
modified: Wed, 03 Feb 2019 00:54:56 -0400
tags:
  - comprhensive
  - TLS
  - PKI
  - openssl
  - gnutls
slug: Trust_TLS
categories:
  - documents
  - articles
  - mutable
  - maintained
featured: true
title: Trust TLS
authors: Kevin Faulkner
summary: An expanding document that should fully describe many components to TLS, geared for quick application
---

#########
Trust TLS
#########
A full document meant to help others with PKI and TLS

******
Trust!
******
What is trust? It really is the common name for the measure of authenticity, mainly testing the location you are trying to connect to. The standard by which something is verified to be genuine. The verification of the client is really more of authentication, user/client. The two are very similar, an authentic which is probably why they both share the same root name, authentikos greek meaning approximately original, primary one. In modern day computing, we do this by trusting a few locations (Root CA's) then they issue trust to Intermediates who can then extend that trust to other locations like \*.lwn.net or a specific name mail.kernel.org. I will try to explain all of this and more. It should be through, and I hope I explain it well. The focus of this document has been geared towards copy and pastable commands, this is so that busy/jr enegineers/administrators can copy and paste and get a mildly working setup. In some places input or specifying a different option is required.

OpenSSL
*******
optional variables, for cleanliness purposes
Generally speaking this will be the style throughout the document
Root-Intermediate-> will be for server certificates
Root-Intermediate-Intermediate-> will be for a cool team, demonstrating that we can have multi-level intermediates. This path is similar to :RFC:`4135` 2.1

Root-IntermediateUsers is User authority
The files will typically have this pattern
Intermediate-${cipher}_,${user},${note}.${type}
Root-${cipher}_${Server},${note}.${type}
Here are the filenames I use, I'm pretty bad at naming things, but something is better than nothing.

.. code-block:: bash

   export CA_PATH=$(pwd)/common_infra
   export CERT_PATH=${CA_PATH}/certs
   export KEY_PATH=${CA_PATH}/keys
   export CSR_PATH=${CA_PATH}/csr
   export SRL_PATH=${CA_PATH}/serial
   export REVOKE_PATH=${CA_PATH}/revoke
   mkdir -p ${CSR_PATH}/ ${KEY_PATH}/ ${CERT_PATH}/ ${REVOKE_PATH}/ ${SRL_PATH}/

RootCA ECC
==========
generate the key, then generate the request, otherwise known as the Certificate signing request. The req request takes the request and immediately signs it and generates the certificate. Root certicates are always self signed, these are the highest level of the trust tree. Feel free to give this certificate out. If you own this machine, distribute it, and trust it (especially in a lab/sandbox).

.. code-block:: bash

   openssl ecparam -genkey -name prime256v1 -out ${KEY_PATH}/RootCA_ECC.key
   openssl req  -new -x509 -sha256 -days 2191 -key ${KEY_PATH}/RootCA_ECC.key -reqexts v3_req -extensions v3_ca -out ${CERT_PATH}/RootCA_ECC.crt 

.. note::   
   Optional, we can do these operations as different commands, if you are learning this might be a good route, just to see what the command above is doing.

.. code-block:: bash

   openssl req -new -sha256 -nodes -key ${KEY_PATH}/RootCA_ECC.key -reqexts v3_req -out ${CSR_PATH}/RootCA_ECC.csr && \
   openssl x509 -req -sha256 -signkey ${KEY_PATH}/RootCA_ECC.key -in ${CSR_PATH}/RootCA_ECC.csr -extensions v3_ca -out ${CERT_PATH}/RootCA_ECC.crt

Optional (generates the revoke list)
------------------------------------
Here a certificate revoke list is being generated. A certificate revoke list a location where clients can check to see if the issuer has revoked the certificate they are about to consume. These are typically not used day-to-day, and OCSP (stapling) is favored. Since OCSP prevents SSL Stripping, which can be done by proxies. If you want something funny, look at the bug of ``man openssl-crl``

.. code-block:: bash

   openssl crl -inform PEM -in ${KEY_PATH}/RootCA_ECC.key -CAfile ${CERT_PATH}/RootCA_ECC.crt -outform DER -out ${REVOKE_PATH}/RootCA_ECC.crl

IntermediateCA ECC
==================
These step could be repeated for a client certificate authority; it might be good so that this CA handles only users and will be need to be explicitly added to which ever trust will be handling the users. Additionally this could be given to an intern or contractor, and if the CA key/secret is compromised, there is limited impact to other CA's.

.. code-block:: bash

   openssl ecparam -genkey -name prime256v1 -out ${KEY_PATH}/IntermediateCA_ECC.key
   openssl req -new -sha256 -nodes -key ${KEY_PATH}/IntermediateCA_ECC.key -out ${CSR_PATH}/IntermediateCA_ECC.csr #CSR's 
   openssl x509 -req -days 1200 -sha256 -in ${CSR_PATH}/IntermediateCA_ECC.csr -CAkey ${KEY_PATH}/RootCA_ECC.key -CA ${CERT_PATH}/RootCA_ECC.crt -out ${CERT_PATH}/IntermediateCA_ECC.crt -CAcreateserial -CAserial ${SRL_PATH}/IntermediateCA_ECC.srl #optional: -set_serial 01

CA Answers
----------
If you want to make an "answers" file, this will allow you to by pass many of OpenSSL's prompts. When dealing with a lot of certificates this is very useful. Here is one for a Certificate Authority. Since by now you've already worked through a prompt with the Root CA, you should pretty much understand what the fields are used for.

| [req]
| prompt = no
| default_md = sha256
| req_extensions = req_ext
| distinguished_name = dn
| [ dn ]
| C=US
| ST=North Carolina
| O=LazyTree
| localityName=
| commonName=*.lazytree.us
| organizationalUnitName=HomeLab
| emailAddress=kondor6c@lazytree.us

Here we'll generate a server certificate with the same encryption type. But we'll do something a little special here. We'll specify extensions to the X509 certificate types. These are added on top of the X509 certificates, the really improve things good deal and chances are you'll need them, almost 100% you'll need Subject Alternate Names, typically just called "SANs". The following is pretty much copy pastable, if you are in a pinch grab and go, replace some of the unique fields like file names and Locality type repsonses.

.. code-block:: bash

   openssl ecparam -genkey -name prime256v1 -out wildcard_lazytree_ECC.pem
   openssl req -new -sha256 -key wildcard_lazytree_ECC.pem -out ${CSR_PATH}/wildcard_lazytree_ECC.csr -config  <(
   cat <<-EOD
   [req]
   prompt = no
   default_md = sha256
   req_extensions = req_ext
   distinguished_name = dn
   [ dn ]
   C=US
   ST=North Carolina
   O=LazyTree
   localityName=Redacted
   OU=HomeLab
   emailAddress=kondor6c@lazytree.us
   CN=null.lazytree.us
   [ req_ext ]
   subjectAltName = @alt_names
   [ alt_names ]
   DNS.1 = expired.lazytree.us
   DNS.2 = testing.lazytree.us
   DNS.3 = lazytree.us
   EOD
   openssl x509 -req -days 800 -sha256 -in ${CSR_PATH}/IntermediateCA_ECC.csr -CAkey IntermediateCA_ECC.pem -CAserial RootCA_ECC.srl -out ${CERT_PATH}/wildcard_lazytree_ECC.crt

Serial
-------
The first time you use your CA to sign a certificate you can use the -CAcreateserial option. This option will create a file (ca.srl) containing a serial number. You are probably going to create more certificate, and the next time you will have to do that use the -CAserial option (and no more -CAcreateserial) followed with the name of the file containing your serial number. This file will be incremented each time you sign a new certificate. This serial number will be readable using a browser (once the certificate is imported to a pkcs12 format). And we can have an idea of the number of certificate created by a CA.

Clients trust
=============
This will allow clients to use certificate in a two manner there are many exampes of big projects that have support of this (but not limited to):
 - postgres
 - dovecot
 - mysql
 - HAProxy
 - Apache
 - nginx
 - curl
 - kafka

I hope the list jogs your mind on where you can take this, two way SSL or "mTLS" or Mutual Authentication is really just allowing the client (the one connecting to the server) to specify a certificate, this is done at the client portion of the TLS handshake, which we'll dig into soon. Let's go ahead and generate the client cert here. I mentioned at the beginning of this documentation that I would try to use a different Intermediate for usage as a client CA. This is because you'll typically need to distribute this CA to clients, and might need to give access to the intermediates to other teams, like a client satisfaction team or sales engineers to issue new client certs quickly. This is just an example, not a best practice.

.. code-block:: bash

   openssl ecparam -genkey -name prime256v1 -out ${client_key_out}
   openssl req -new -sha256 -key ${client_key_out} -out ${CSR_PATH}/client_lazytree_ECC.csr
   openssl x509 -req -days 300 -sha256 -in ${CSR_PATH}/client_lazytree_ECC.csr -CA ${CERT_PATH}/  IntermediateClientCA_ECC.crt -CAkey IntermediateClientCA_ECC.pem -out ${CERT_PATH}/client_lazytree_ECC.crt

Chains or Bundles
-----------------
Chains can be used, or they don't have to be. The usage lies in the fact that if an intermediate is not trusted, but the root certificate is, or another intermediate in the chain is trusted. The name bundles are used because there are bundles of certificates (Root and Intermediates), it is *highly recommended* that the fully chain, be sent (hey you reading this, send dat chain!). You can find options that are used for CA Chains in the server secition below. The order is defined in `RFC-5246 <https://tools.ietf.org/html/rfc5246#section-7.4.2>` The order is exactly as follows:
 1. Server Certificate
 2. Intermediate
 3. <optional> another Intermediate that has signed number two
 4. Root Certificate

::

   cat IntermediateCA.crt RootCA.crt > Cert-Chain.pem
   cat IntermediateCA_ECC.crt RootCA_ECC.crt > Cert-Chain_ECC.pem

Verification of Certificates
============================
It is always good to verify your work, even better to have a buddy check your work too, you never know what you might learn from somebody else's perspective.

Examine a certificate
---------------------
Check your work

::

  openssl x509 -in ${CERT_PATH}/certificate.crt -text -noout

Examine a key (RSA)
^^^^^^^^^^^^^^^^^^^
You can also look at the key you produced

::

   openssl rsa -in privateKey.key -check

Examine a Certificate Signing Request (CSR)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To view a previously generated certificate signing request you can run the following.

::

   openssl req -text -noout -verify -in CSR.csr

Revoke a Cert
-------------
As mentioned, revokation lists and the revoking process isn't done too much. But it could really help out, consider an example, 24 hours before a certificate is about to expire if an Internal CA were to revoke the soon to expire certificate, you will have an opportunity to know for sure which applciations depend on the certifcate. This could be very useful for large organizations. Just a tip!

::

   openssl ca -config ca.conf -revoke ia.crt -keyfile ca.key -cert ${CERT_PATH}/ca.crt -crl_reason superseded

Configuring SSL on Operating Systems
************************************
Here is a list of operating systems and how to configure SSL on them, I hope this helps, if you know of somelet me know (open a pull request).

Windows
=======
First we need to prep, the the best of my knowledge windows doesn't handle pem formats, which is pretty frustrating. So we need to export it to a PKCS12 format.

.. code-block:: bash

   openssl pkcs12 -export -in wildcard_lazytree_ECC.crt -inkey wildcard_lazytree_ECC.pem -out wildcard_lazytree_ECC.pfx -certfile Cert-Chain_ECC.pem
   openssl pkcs12 -export -in wildcard_lazytree.crt -inkey wildcard_lazytree.pem -out wildcard_lazytree.pfx -certfile  Cert-Chain.pem
   openssl pkcs12 -export -nokeys -in RootCA.crt -out RootCA.pfx
   openssl pkcs12 -export -nokeys -in RootCA_ECC.crt -out RootCA_ECC.pfx

Now we can take that file and add it to Windows

.. code-block:: console

   certutil.exe -addstore "RootCA_SHA1" RootCA.pfx
   certutil.exe -addstore "RootCA_ECC" RootCA_ECC.pfx
   certutil.exe -importPFX wildcard_lazytree_ECC.pfx
   certutil.exe -importPFX wildcard_lazytree.pfx

RHEL-like Linux
===============
You can easily add certificates to Redhat like distributions like Fedora, Centos, Amazon Linux, Scientific Linux or Oracle Linux. Consider distributing this as an RPM.

.. code-block:: bash

   rsync -va \*crt /etc/pki/ca-trust/source/anchors/
   update-ca-trust force-enable

Debian-like Linux AND Gentoo
============================
`Gentoo <https://wiki.gentoo.org/wiki/Certificates>` https://www.archlinux.org/news/ca-certificates-update/

.. code-block:: bash

   rsync -va \*crt /usr/local/share/ca-certificates/
   update-ca-certificates

Android
=======
Settings > Security & Lock Screen > Credential storage (under "advanced") > Install from storage

Applications
************
Java
====
Java holds the keys and certificates in a special file, called a keystore. It used to be a proprietary format JKS, but the newer, preferred format is p12 (PKCS12). You can access it with keytool, which should be in the same path as ``java`` ($JAVA_HOME/bin/).

::

  keytool -v -list -keystore /etc/pki/ca-trust/extracted/java/cacert || keytool -v -list -keystore /etc/pki/java/cacerts #changeit is Java's default
  keytool -import -trustcacerts -alias rootCA_ECC -file  RootCA_ECC.crt
  keytool -import -trustcacerts -alias IntermediateCA_ECC -file  IntermediateCA_ECC.crt
  keytool -import -trustcacerts -alias rootCA_weak -file  RootCA.crt
  keytool -import -trustcacerts -alias IntermediateCA_weak -file  IntermediateCA.crt

Chrome
======
https://stackoverflow.com/questions/7580508/getting-chrome-to-accept-self-signed-localhost-certificate
You can avoid the message for trusted sites by installing the certificate.
This can be done by clicking on the warning icon in the address bar, then click
"Not secure" -> Certificate Invalid -> Details Tab -> Export... Save the certificate.

Use Chrome's Preferences -> Under The Hood -> Manage Certificates -> Import.
On the "Certificate Store" screen of the import, choose "Place all certificates in the following store" and browse for "Trusted Root Certification Authorities." Restart Chrome.
Chrome Settings > Show advanced settings > HTTPS/SSL > Manage Certificates > Authorities

Nginx
=====
https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7

| server {
|     listen 80;
|     server_name "example.lazytree.us";
|     return 301 https://$host$request_uri;
| }
| server {
|     server_name "10.1.1.1"
|     listen 443 http2 ssl;
|     listen [::]:443 http2 ssl;
|     ssl_certificate /etc/ssl/certificates/example.lazytree.us/app_role.crt;
|     ssl_certificate_key /etc/ssl/keys/example.lazytree.us/app_role.key;
|     ssl_dhparam /etc/ssl/keys/example.lazytree.us/dhparam.pem;
| }

Apache
======
https://wiki.apache.org/httpd/RedirectSSL

| Listen 443 ssl
| <VirtualHost _default_:443>
|  ServerName lazytree.us
|  SSLEngine on
|  SSLProtocol all -SSLv2 -SSLv3
|  SSLCertificateFile /etc/pki/tls/certs/
|  SSLCertificateKeyFile /etc/pki/tls/private/
|  SSLCertificateChainFile /etc/pki/tls/certs/chain.crt
|  SSLCACertificateFile /etc/httpd/conf.d/tls/client_IntermediateCA.crt
|  SSLOpenSSLConfCmd DHParameters "/etc/pki/ssl/dhparams.pem"
|  RewriteEngine On 
|  RewriteCond %{HTTPS} off
|  RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R,L]
| </VirtualHost>

# It would be nice to get blake2s256 supported in more places
#GPG fingerprint = 7545BFF3710684D2E6BCFE98C5D5F4BED24A4A02
#GPG fingerprint = 438263E03BF0BDC64F9A6415AA63E0576CC60292



GNU TLS
*******
I have recently been liking GnuTLS since it has rather descriptive options, they are easy to read and self describing of the process. The issue is that it isn't always installed.

.. code-block:: console

   certtool --generate-privkey --bits 4096 --outfile RootCA_G-RSA.pem
   certtool --generate-request --load-privkey RootCA_G-RSA.pem --hash=SHA256 --template gnutls-ssl-answers.txt --outfile RootCA_G-RSA.csr
   certtool --generate-certificate --load-privkey RootCA_G-ECC.pem --outfile RootCA_G-ECC.crt --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem 

GNU TLS ECC
===========
Coming in version 3.6!! Ed25519 keys
``certtool --generate-privkey --key-type ed25519 --outfile RootCA_G-ECC.pem`` 
otherwise you might need to go with secp 256

.. code:: bash

   certtool --generate-privkey --ecc --curve secp256r1 --outfile RootCA_G-ECC.pem
   certtool --generate-request --load-privkey RootCA_G-ECC.pem --hash=SHA256 --outfile RootCA_G-ECC.csr
   certtool --generate-certificate --load-privkey RootCA_G-ECC.pem --outfile RootCA_G-ECC.crt --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem 


Quick Reference Lists
*********************
You could call these "cheat sheets" but these are more translation matrixes, like a rosetta stone of options. I often get frustrated with how many different options I'm called to remember (not entirely told, but just feel as though, professionally, I should). It can be difficult not specializing in a specific peice of software, since you have a ever expanding target of defaults, types of actions, configurations locations, and command line arguements; but I digress. I hope to make more of these, checkout my dotfiles where I use the top 26 lines as a quick reference. Some the options I know, others I have a hard time remembering, while others I learned while making it. https://github.com/kondor6c/dotfiles

verification
************
+-----------------------+-----------------------------+------------------------+
| OpenSSL               | GnuTLS                      | function               |
+=======================+=============================+========================+
| x509 -verify          | --verify                    | verify x509 cert       |
+-----------------------+-----------------------------+------------------------+
| x509 -CAfile          | --load-ca-certificate       | verify chain CA file   |
+-----------------------+-----------------------------+------------------------+
| x509 -text -noout -in | --certificate-info --infile | verify an x509 cert    |
+-----------------------+-----------------------------+------------------------+
| req -noout -text -in  | --crq-info --infile         | examine a CSR          |
+-----------------------+-----------------------------+------------------------+

x509 generation
***************
+------------------------+-------------------------+---------------------------+
| OpenSSL                | GnuTLS                  | function                  |
+========================+=========================+===========================+
| x509 -text -noout -in  | --verify --infile       | verify x509 certicate     |
+------------------------+-------------------------+---------------------------+
| x509 -CAfile           | --load-ca-certificate   | verify chain againt file  |
+------------------------+-------------------------+---------------------------+
| x509 -CAkey            | --load-ca-privkey       | load CA key to sign       |
+------------------------+-------------------------+---------------------------+
| x509 -req              | --load-request          | load CSR to sign for cert |
+------------------------+-------------------------+---------------------------+
| x509 -CA               | --load-ca-certificate   | Load CA cert to sign      |
+------------------------+-------------------------+---------------------------+
| x509 -config           | --template              | preconfigured answers     |
+------------------------+-------------------------+---------------------------+
| x509 -sha256           | --hash=SHA256           | certificate hash (sha256) |
+------------------------+-------------------------+---------------------------+


keys (RSA)
**********
+----------------------+------------------------------+------------------------+
| OpenSSL              | GnuTLS                       | function               |
+======================+==============================+========================+
| genrsa -out          | --generate-privkey --outfile | Write rsa Key to file  |
+----------------------+------------------------------+------------------------+
| rsa -noout -text -in | -k --infile                  | examine RSA Key        |
+----------------------+------------------------------+------------------------+

Diffie Helman
*************
+-------------------+------------------------------+------------------------+
| OpenSSL           | GnuTLS                       | function               |
+===================+==============================+========================+
| dhparam 2048 -out | --generate-dh-params         | generate parameters    |
+-------------------+------------------------------+------------------------+

ref
***
links and stuff
https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html

RFC list
********

Forward Secrecyree_SSH_CA-ed -h -t ed25519 -a 100 
ssh-keygen -f LazyTree_SSH_CA-ecdsa -h -t ecdsa -b 521 
ssh-keygen -f LazyTree_SSH_CA-rsa -h -t rsa -b 4096

Traditional Host Keys
=====================
ssh-keygen -f LazyTree_SSH-rsa -t rsa -b 4096
ssh-keygen -f LazyTree_SSH-ed -t ed25519 -a 100
ssh-keygen -f LazyTree_SSH-ecdsa -t ecdsa -b 521


DH Params
=========
Diffie Helman is pretty cool

::

  openssl dhparam -out dhparam.pem 4096

OpenSSH CA
**********
https://blog.habets.se/2011/07/OpenSSH-certificates.html

Host CA's
*********
ssh-keygen -f LazyTree_SSH_CA-ed -h -t ed25519 -a 100 
ssh-keygen -f LazyTree_SSH_CA-ecdsa -h -t ecdsa -b 521 
ssh-keygen -f LazyTree_SSH_CA-rsa -h -t rsa -b 4096

Traditional Host Keys

ssh-keygen -f LazyTree_SSH-rsa -t rsa -b 4096
ssh-keygen -f LazyTree_SSH-ed -t ed25519 -a 100
ssh-keygen -f LazyTree_SSH-ecdsa -t ecdsa -b 521


