# Reed-Solomon Error Correction Codes

**Original view and BCH view**

There are two views of Reed-Solomon codes, make sure which _view_ you are reading about.

## Reference

* <https://en.wikipedia.org/wiki/Reed-Solomon_error_correction>

### Tutorials

### Videos

* [vcubingx](https://vcubingx.com) - [What are Reed-Solomon Codes?](https://www.youtube.com/watch?v=1pQJkt7-R4Q)
* [Isabel Vogt](https://www.math.brown.edu/ivogt/) - [Error Correcting Curves - Numberphile](https://www.youtube.com/watch?v=CcZf_7Fb4Us)
* [Mary Wootters](https://sites.google.com/site/marywootters/home) - [Lecture 4: Reed-Solomon Codes!](https://www.youtube.com/watch?v=yQkEnde2lNg) of Stanford [CS250/EE387: Algebraic Error Correcting Codes](https://web.stanford.edu/class/cs250/)
    * [Her 1-hr long public lecture for wider audience](https://www.youtube.com/watch?v=xE4jEKx9fTM&ab_channel=SimonsInstitute)

## Open source

### Original view (Vandermonde Reed-Solomon)

* James S. Plank: [Tutorial Material for Erasure Coding](http://web.eecs.utk.edu/~plank/plank/www/software.html), [SPE97 paper](http://cgi.di.uoa.gr/~ad/M155/Papers/RS-Tutorial.pdf) and [correction 2003](https://web.eecs.utk.edu/~jplank/plank/papers/SPE-04.html).

* Luigi Rizzo: [Software FEC in computer communications](http://info.iet.unipi.it/~luigi/fec.html), [RFC 5510](https://www.rfc-editor.org/rfc/rfc5510.html)
* [JavaReedSolomon](https://github.com/Backblaze/JavaReedSolomon) by Backblaze.
    * Blog post [Backblaze Open-sources Reed-Solomon Erasure Coding Source Code](https://www.backblaze.com/blog/reed-solomon/) and [5-minute video tutorial by Brian Beach](https://www.youtube.com/watch?v=jgO09opx56o&ab_channel=Backblaze)
* Python [pyfinite](https://github.com/emin63/pyfinite) package.

### BCH view

* [Error Correcting Codes (ECC) Page](https://www.eccpage.com/) by Robert Morelos-Zaragoza.
* [FEC Library](https://www.ka9q.net/code/fec/) by Phil Karn, KA9Q.
    * Adapted into [Linux kernel](https://github.com/torvalds/linux/tree/master/lib/reed_solomon) 2.6.10 by Thomas Gleixner.
* Russ Cox blog [Finite Field Arithmetic and Reed-Solomon Coding](https://research.swtch.com/field) with [source code in Go](https://github.com/rsc/qr/tree/master/gf256).
* Python [galois](https://mhostetter.github.io/galois/latest/) package.

## Erasure codes in Python

### Finite Field (Galois Field)

```
```

## History


![Authors](reed-solomon.jpg)
