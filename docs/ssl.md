# Notes on SSL/TLS

* [RFC7539](https://tools.ietf.org/html/rfc7539) defines ChaCha20 and Poly1305
* [RFC7905](https://tools.ietf.org/html/rfc7905) adds TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xCCA8) etc to TLS 1.2
* There is an old version of ECDHE_RSA_CHACHA20_POLY1305_OLD (0xCC13) exists in [draft](https://tools.ietf.org/html/draft-agl-tls-chacha20poly1305) and OpenSSL [code](https://boringssl.googlesource.com/boringssl/+/271777f5ac6c7f7d1aaf95b91c5a5bef86da6363).
* OpenSSL will prefer AES if hardware supports it, otherwise prefer ChaCha20 over AES, in [ssl_create_cipher_list()](https://github.com/libressl-portable/openbsd/blob/master/src/lib/libssl/ssl_ciph.c#L1397).
* OpenSSL 1.1.0 provides [SSL_SESSION_get_master_key()](https://www.openssl.org/docs/man1.1.0/ssl/SSL_get_client_random.html) function.
* Before OpenSSL 1.1.0, there is a free-list (~32k per SSL) unless disable with OPENSSL_NO_BUF_FREELISTS or set SSL_MODE_RELEASE_BUFFERS, the free list is freed in `SSL_CTX_free()` not `SSL_free()`.

[commit 63c574f6](https://github.com/openssl/openssl/commit/63c574f6a639cfa3f53476080054526e6bfa3bc9) in OpenSSL:

```
commit 63c574f6a639cfa3f53476080054526e6bfa3bc9
Author: Rich Salz <rsalz@openssl.org>
Date:   Tue Jan 27 16:43:53 2015 -0500

    OPENSSL_NO_XXX cleanup: OPENSSL_NO_BUF_FREELISTS

    Remove OPENSSL_NO_BUF_FREELISTS. This was turned on by default,
    so the work here is removing the 'maintain our own freelist' code.
    Also removed a minor old Windows-multibyte/widechar conversion flag.
```

[commit 72e56bc4](https://github.com/libressl-portable/openbsd/commit/72e56bc4947e53cec535f2fc84e00a3c278edf29) in LibreSSL:

```
commit 72e56bc4947e53cec535f2fc84e00a3c278edf29
Author: tedu <>
Date:   Wed Apr 16 21:16:33 2014 +0000

    TANSTAAFL - delete the buf freelist code. if you need a better malloc, get
    a better malloc. ok beck deraadt

diff --git a/src/lib/libssl/s3_both.c b/src/lib/libssl/s3_both.c
```
[TANSTAAFL - There ain't no such thing as a free lunch](https://en.wikipedia.org/wiki/There_ain't_no_such_thing_as_a_free_lunch)

[commit 983f6bdb](https://boringssl.googlesource.com/boringssl/+/983f6bdb5888d7074d7ee67d5e3741bcf1027108%5E%21/) in BoringSSL:

```
commit 983f6bdb5888d7074d7ee67d5e3741bcf1027108
Author: Alex Chernyakhovsky <achernya@google.com>
Date:   Sun Aug 3 15:48:35 2014 -0400

    Set OPENSSL_NO_BUF_FREELISTS

    The memory freelist maintained by OpenSSL claims to be a performance
    optimization for platforms that have a slow malloc/free
    implementation. This should not be the case on modern
    linux/glibc. Remove the freelist as it poses a potential security
    hazard of buffer-reuse that is of "initialized" memory that will not
    be caught be tools such as valgrind.

    Change-Id: I3cfa6a05f9bdfbbba7820060bae5a673dee43014
    Reviewed-on: https://boringssl-review.googlesource.com/1385
    Reviewed-by: Adam Langley <agl@google.com>
```

* After [commit b8d28cf532](https://boringssl.googlesource.com/boringssl/+/b8d28cf532e6dde3d5684ededb3878b05e692e00) BoringSSL calls `ssl_write_buffer_flush()` and `ssl_write_buffer_clear()` for each `SSL_write()`, so its memory footprint is much smaller than other SSL libraries (~32k) when no data is pending.
