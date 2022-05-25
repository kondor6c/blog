---
title: GnuTLS and ed25519
date: 2018-08-19
tags:
  - linux
  - certs
  - gnutls
slug: gnutls_ed25519
description: GNUTLS 3.6 brings ed25519 key support
---

I'm very excited that GNUTLS has support for ed25519 keys, these keys are faster, smaller, and more secure than RSA keys. I found this out from a developer's blog while doing research and messing around with CA's in December. The reason I'm writing this is that GNUTLS 3.6 has been made available in Fedora 28, therefore can all use this. Additionally and perhaps more importantly we gain the the ability to use TLS 1.3, which cloudflare has [written]_ [about]_. The arrival of this support is slightly bitter sweet because we do lose support for GPG based x509 [certificates]_. This is unfortunate in principle because the CA's that are apporved can impose a signifigant cost to an individual, I certainly have not used GPG based x509 certificates. I don't believe this is due an issue with lack of interest, I believe it is due to the difficulty of GPG and the fact that the end user often needs to have some kind of knowledge of PKI and how GPG does it; which I have found to be lacking. But like the GNUTLS developer [wrote]_, we can use Let's Encrypt, but it is hard to say what might be in Let's Encrypt's future. I sincerely wish it the best, it is nice to have competing or overlapping technologies, having the extra functionality does pose a couple issues because it increases an attack vector and more maintenance would be required on the software. I haven't found many people to use GNUTLS, but maybe people might with Kubnetes (additionally there have been concerns with OpenSSL). I guess we have that and we can roll our own CA! More on that later!

We should briefly look at creating an ed25519 key with GNUTLS:
``certtool --generate-privkey --key-type ed25519 --outfile key-ed25519.pem``
That's it! Really simple, and personally I think it is much easier and readable than OpenSSL.

.. [written] https://blog.cloudflare.com/why-iot-is-insecure/
.. [about] https://blog.cloudflare.com/you-get-tls-1-3-you-get-tls-1-3-everyone-gets-tls-1-3/
.. [certificates] https://www.gnutls.org/manual/html_node/OpenPGP-certificates.html
.. [wrote] https://nikmav.blogspot.com/2017/
