# Notes on SSL/TLS

* [RFC7539](https://tools.ietf.org/html/rfc7539) defines ChaCha20 and Poly1305
* [RFC7905](https://tools.ietf.org/html/rfc7905) adds TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xCCA8) etc to TLS 1.2
* There is an old version of ECDHE_RSA_CHACHA20_POLY1305_OLD (0xCC13) exists in [draft](https://tools.ietf.org/html/draft-agl-tls-chacha20poly1305) and OpenSSL [code](https://boringssl.googlesource.com/boringssl/+/271777f5ac6c7f7d1aaf95b91c5a5bef86da6363).
* OpenSSL will prefer AES if hardware supports it, otherwise prefer ChaCha20 over AES, in [ssl_create_cipher_list()](https://github.com/libressl-portable/openbsd/blob/master/src/lib/libssl/ssl_ciph.c#L1397).
* OpenSSL 1.1.0 provides [SSL_SESSION_get_master_key()](https://www.openssl.org/docs/man1.1.0/ssl/SSL_get_client_random.html) function.
