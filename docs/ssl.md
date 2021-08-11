# Notes on SSL/TLS

## ChaCha20, Poly1305, X25519
* [RFC7539](https://tools.ietf.org/html/rfc7539) defines ChaCha20 and Poly1305
* [RFC7905](https://tools.ietf.org/html/rfc7905) adds TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xCCA8) etc to TLS 1.2
* There is an old version of ECDHE_RSA_CHACHA20_POLY1305_OLD (0xCC13) exists in [draft](https://tools.ietf.org/html/draft-agl-tls-chacha20poly1305) and OpenSSL [code](https://boringssl.googlesource.com/boringssl/+/271777f5ac6c7f7d1aaf95b91c5a5bef86da6363).
* OpenSSL will prefer AES if hardware supports it, otherwise prefer ChaCha20 over AES, in [ssl_create_cipher_list()](https://github.com/libressl-portable/openbsd/blob/master/src/lib/libssl/ssl_ciph.c#L1397).
* Go 1.8 will support TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 [commit 67d8226b48](https://go.googlesource.com/go/+/67d8226b4862f0f8deb4dc6fa8617017ecb0f32b)

```text
commit 67d8226b4862f0f8deb4dc6fa8617017ecb0f32b
Author: Adam Langley <agl@golang.org>
Date:   Wed Oct 12 10:46:43 2016 -0700

    crypto/tls: support ChaCha20-Poly1305.

    This change adds support for the ChaCha20-Poly1305 AEAD to crypto/tls,
    as specified in https://tools.ietf.org/html/rfc7905.

    Fixes #15499.

    Change-Id: Iaa689be90e03f208c40b574eca399e56f3c7ecf1
    Reviewed-on: https://go-review.googlesource.com/30957
    Run-TryBot: Adam Langley <agl@golang.org>
    TryBot-Result: Gobot Gobot <gobot@golang.org>
    Reviewed-by: Brad Fitzpatrick <bradfitz@golang.org>
```

* Go perfers X25519 over P-256, P-384 and P-521. [commit 9e98e7e668](https://go.googlesource.com/go/+/9e98e7e6686a500f87352e3721038d194b1db33c)

```text
commit 9e98e7e6686a500f87352e3721038d194b1db33c
Author: Adam Langley <agl@golang.org>
Date:   Tue Oct 11 15:08:35 2016 -0700

    crypto/tls: enable X25519 by default.

    Since this changes the offered curves in the ClientHello, all the test
    data needs to be updated too.

    Change-Id: I227934711104349c0f0eab11d854e5a2adcbc363
    Reviewed-on: https://go-review.googlesource.com/30825
    Reviewed-by: Brad Fitzpatrick <bradfitz@golang.org>
```

## API changes

* OpenSSL 1.0.2 has an undocumented `SSL_get_server_tmp_key()` function to get the curve used in handshaking, this is for `openssl s_client` command. <http://crypto.stackexchange.com/questions/11310/with-openssl-and-ecdhe-how-to-show-the-actual-curve-being-used>
* OpenSSL 1.0.1 only allows setting one curve with `SSL_CTX_set_tmp_ecdh()` (or use callbacks), OpenSSL 1.0.2 allows setting curve list with `SSL_CTX_set1_curves()` and `SSL_CTX_set1_curves_list()`
* OpenSSL 1.0.2 adds `SSL_CTX_set_ecdh_auto()`, but OpenSSL 1.1.0 makes it a no-op.
* OpenSSL 1.1.0 provides [SSL_SESSION_get_master_key()](https://www.openssl.org/docs/man1.1.0/ssl/SSL_get_client_random.html) function.

* Go 1.8 supports KeyLogWriter [commit 320bd562cb](https://go.googlesource.com/go/+/320bd562cbb24a01beb02706c42d06a290160645)

```text
commit 320bd562cbb24a01beb02706c42d06a290160645
Author: Joonas Kuorilehto <joneskoo@derbian.fi>
Date:   Sat Aug 20 14:41:42 2016 +0300

    crypto/tls: add KeyLogWriter for debugging

    Add support for writing TLS client random and master secret
    in NSS key log format.

    https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/Key_Log_Format

    Normally this is enabled by a developer debugging TLS based
    applications, especially HTTP/2, by setting the KeyLogWriter
    to an open file. The keys negotiated in handshake are then
    logged and can be used to decrypt TLS sessions e.g. in Wireshark.

    Applications may choose to add support similar to NSS where this
    is enabled by environment variable, but no such mechanism is
    built in to Go. Instead each application must explicitly enable.

    Fixes #13057.

    Change-Id: If6edd2d58999903e8390b1674ba4257ecc747ae1
    Reviewed-on: https://go-review.googlesource.com/27434
    Run-TryBot: Brad Fitzpatrick <bradfitz@golang.org>
    TryBot-Result: Gobot Gobot <gobot@golang.org>
    Reviewed-by: Brad Fitzpatrick <bradfitz@golang.org>
```

## Memory footprint

* Before OpenSSL 1.1.0, there is a free-list (~32k per SSL) unless disable with OPENSSL_NO_BUF_FREELISTS or set SSL_MODE_RELEASE_BUFFERS, the free list is freed in `SSL_CTX_free()` not `SSL_free()`.

[commit 63c574f6a6](https://github.com/openssl/openssl/commit/63c574f6a639cfa3f53476080054526e6bfa3bc9) in OpenSSL:

```text
commit 63c574f6a639cfa3f53476080054526e6bfa3bc9
Author: Rich Salz <rsalz@openssl.org>
Date:   Tue Jan 27 16:43:53 2015 -0500

    OPENSSL_NO_XXX cleanup: OPENSSL_NO_BUF_FREELISTS

    Remove OPENSSL_NO_BUF_FREELISTS. This was turned on by default,
    so the work here is removing the 'maintain our own freelist' code.
    Also removed a minor old Windows-multibyte/widechar conversion flag.
```

[commit 72e56bc494](https://github.com/libressl-portable/openbsd/commit/72e56bc4947e53cec535f2fc84e00a3c278edf29) in LibreSSL:

```text
commit 72e56bc4947e53cec535f2fc84e00a3c278edf29
Author: tedu <>
Date:   Wed Apr 16 21:16:33 2014 +0000

    TANSTAAFL - delete the buf freelist code. if you need a better malloc, get
    a better malloc. ok beck deraadt

diff --git a/src/lib/libssl/s3_both.c b/src/lib/libssl/s3_both.c
```
[TANSTAAFL - There ain't no such thing as a free lunch](https://en.wikipedia.org/wiki/There_ain't_no_such_thing_as_a_free_lunch)

[commit 983f6bdb58](https://boringssl.googlesource.com/boringssl/+/983f6bdb5888d7074d7ee67d5e3741bcf1027108%5E%21/) in BoringSSL:

```text
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

## Performance notes

The P-256 curve, also known as `prime256v1` or `secp256r1` is widely used in ECDHE_RSA key exchange.

For x86-64, there is a faster implementation, enabled by `./config enable-ec_nistp_64_gcc_128`.

Futher speed up of P-256 in OpenSSL 1.1.0 and BoringSSL:

* [commit 4d3fa06fce](https://github.com/openssl/openssl/commit/4d3fa06fce52682bfbc503c7ded2d0289e3f8cde) for OpenSSL 1.1.0
* [commit 3842a64d36](https://github.com/openssl/openssl/commit/3842a64d36bc47ef7cc1370147fd0e5b60a27a2b) backports to 1.0.2 branch
* [commit 1895493868](https://boringssl.googlesource.com/boringssl/+/18954938684e269ccd59152027d2244040e2b819) for BoringSSL

```text
commit 18954938684e269ccd59152027d2244040e2b819
Author: Adam Langley <agl@google.com>
Date:   Tue Nov 3 14:02:04 2015 -0800

    Add Intel's P-256

    This change incorporates Intel's P-256 implementation. The record of
    Intel's submission under CLA is in internal bug number 25330687.

    Before:
    Did 3582 ECDH P-256 operations in 1049114us (3414.3 ops/sec)
    Did 8525 ECDSA P-256 signing operations in 1028778us (8286.5 ops/sec)
    Did 3487 ECDSA P-256 verify operations in 1008996us (3455.9 ops/sec)
    build/tool/bssl is 1434704 bytes after strip -s

    After:
    Did 8618 ECDH P-256 operations in 1027884us (8384.2 ops/sec)
    Did 21000 ECDSA P-256 signing operations in 1049490us (20009.7 ops/sec)
    Did 8268 ECDSA P-256 verify operations in 1079481us (7659.2 ops/sec)
    build/tool/bssl is 1567216 bytes after strip -s

    Change-Id: I147971a8e19849779c8ed7e20310d41bd4962299
    Reviewed-on: https://boringssl-review.googlesource.com/6371
    Reviewed-by: Adam Langley <agl@google.com>
```

None of them are available in LibreSSL as of Jan 19, 2017. [Issue 259](https://github.com/libressl-portable/portable/issues/259).

## AEGIS-128 will replace AES-GCM

One of winners of [CAESAR cryptographic competitions](http://competitions.cr.yp.to/caesar-submissions.html).

## Cryptography

* [cs387 Applied Cryptography on udacity](https://www.udacity.com/course/applied-cryptography--cs387)
* [Elliptic Curve Cryptography: ECDH and ECDSA](http://andrea.corbellini.name/2015/05/30/elliptic-curve-cryptography-ecdh-and-ecdsa/)
* [Everything you need to know about cryptography in 1 hour](http://www.bsdcan.org/2010/schedule/attachments/135_crypto1hr.pdf)

