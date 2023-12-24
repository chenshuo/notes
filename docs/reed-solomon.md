# Reed-Solomon Error Correction Codes

There are two views of Reed-Solomon codes, **Original view and BCH view**, make sure which _view_ you are reading about.

It seems to me, BCH view is most common used (QR codes, CD/DVD, FEC in communications like DVB or 100GbE), and can do both erasure and error corrections. Original view is mainly used for erasure codes in (distributed) storage systems.

| | Original view | BCH view |
| --- | --- | --- |
| Message \(\boldsymbol m = [m_0, m_1, \ldots, m_{k-1}]\)  <td colspan=2>As coefficients of polynomial \(p(x) = m_0 + m_1x + m_2x^2 + \cdots + m_{k-1}x^{k-1}\)
| Codeword \(\boldsymbol c = [c_0, c_1, \ldots, c_{n-1}]\) | As values of \(p(x)\) at \(a_0, a_1, \ldots, a_{n-1}\)| As coefficients of polynomial \(s(x)\) |
| Characteristic    | Vandermonde matrix \(V\) | Generator polynomial \(g(x)\)|
| \((n, k)\) | Usuall small, for erasure codes in storage systems | \(n=2^m-1\), usually \(n=255\) |
| Basic encoding | \(V_{n \times k} \cdot \boldsymbol m \) | \(s(x) = p(x) \cdot g(x)\) |  |
| Systematic encoding | $$\begin{bmatrix}I_k\\G\end{bmatrix} \cdot \boldsymbol m$$  | \(p(x) \cdot x^{n-k} \mod g(x)\) |
| Implementation | More memory for \(V\) or \(G\) | linear feedback shift register (LFSR)  |
| Erasure decoding | \(M_k^{-1}\) (Inverse of matrix)| Solve linear equations? |
| Error correction decoding| Berlekamp-Welch (1983), Shuhong Gao (2002) | PGZ (1960), Berlekamp-Massey (1967)  |  |

$$V_{n\times k} = \begin{pmatrix}  1 & a_0 & a_0^2 & \cdots & a_0^{k-1} \\[1mm]  1 & a_1 & a_1^2 & \cdots & a_1^{k-1} \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 1 & a_{n-1} & a_{n-1}^2 & \cdots & a_{n-1}^{k-1} \end{pmatrix}, \quad g(x) = (x-\alpha^i)(x-\alpha^{i+1})\cdots (x-\alpha^{i+n-k-1})$$


## Erasure codes in Python

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/chenshuo/notes/blob/master/notebooks/ReedSolomonErasureCodes.ipynb)

<https://github.com/chenshuo/notes/blob/master/notebooks/ReedSolomonErasureCodes.ipynb>

## Reference

* <https://en.wikipedia.org/wiki/Reed-Solomon_error_correction>

### Tutorials

**Starts here**

* "Practical Reed-Solomon for Programmers" <https://berthub.eu/articles/posts/reed-solomon-for-programmers/> (BCH view)
* <https://www.akalin.com/intro-erasure-codes> (Original view)

**Polynomial visualize**

* "Reed-Solomon Error Correcting Codes from the Bottom Up" <https://tomverbeure.github.io/2022/08/07/Reed-Solomon.html>
* "Introduction to Reed-Solomon" <https://innovation.vivint.com/introduction-to-reed-solomon-bc264d0794f8>
    * <https://github.com/vivint/infectious>

**Finite Field**

* "Finite Field Arithmetic and Reed-Solomon Coding" <https://research.swtch.com/field> (BCH view)
    * <https://github.com/rsc/qr/blob/master/gf256/gf256.go>
* "The mathematics of RAID-6" <https://www.kernel.org/pub/linux/kernel/people/hpa/raid6.pdf> (Finite Field Arithmetic)

**Error correction decoding in BCH view**

* [Reed–Solomon codes for coders](https://en.wikiversity.org/wiki/Reed%E2%80%93Solomon_codes_for_coders) (BCH view with Python code)
* <https://downloads.bbc.co.uk/rd/pubs/whp/whp-pdf-files/WHP031.pdf> (BCH view)
* <https://www.nayuki.io/page/reed-solomon-error-correcting-code-decoder> (BCH view, PGZ decoder)


**Erasure decoding in original view**

* Erasure Codes for Storage Applications <https://web.eecs.utk.edu/~jplank/plank/papers/FAST-2005.pdf>
* Tutorial: Erasure Coding for Storage Applications <https://web.eecs.utk.edu/~jplank/plank/papers/FAST-2013-Tutorial.html>
* All About Erasure Codes by James Plank <https://web.eecs.utk.edu/~jplank/plank/classes/cs560/560/notes/Erasure/2004-ICL.pdf>

### Videos

* [vcubingx](https://vcubingx.com) - [What are Reed-Solomon Codes?](https://www.youtube.com/watch?v=1pQJkt7-R4Q)
* [Isabel Vogt](https://www.math.brown.edu/ivogt/) - [Error Correcting Curves - Numberphile](https://www.youtube.com/watch?v=CcZf_7Fb4Us)
* [Mary Wootters](https://sites.google.com/site/marywootters/home) - [Lecture 4: Reed-Solomon Codes!](https://www.youtube.com/watch?v=yQkEnde2lNg) of Stanford [CS250/EE387: Algebraic Error Correcting Codes](https://web.stanford.edu/class/cs250/)
    * [Her 1-hr long public lecture for wider audience](https://www.youtube.com/watch?v=xE4jEKx9fTM&ab_channel=SimonsInstitute)

### Course notes
* Stanford CS250/EE387: Algebraic Error Correcting Codes <https://web.stanford.edu/class/cs250/>
* Berkeley CS 294-226: Advances in Error-Correcting Codes <https://people.eecs.berkeley.edu/~venkatg/teaching/ECC-fall22/>
* MIT 6.451: Principles Of Digital Communication II <https://ocw.mit.edu/courses/6-451-principles-of-digital-communication-ii-spring-2005/resources/mit6_451s05_fulllecnotes/>

## Open source

### Original view (Vandermonde Reed-Solomon)

* James S. Plank: [A Tutorial on Reed-Solomon Coding for Fault-Tolerance in RAID-like Systems](https://web.eecs.utk.edu/~jplank/plank/papers/SPE-9-97.html), [SPE97 paper](http://cgi.di.uoa.gr/~ad/M155/Papers/RS-Tutorial.pdf) and [correction 2003](https://web.eecs.utk.edu/~jplank/plank/papers/SPE-04.html).

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

## History and People

![Authors](reed-solomon.jpg)

* <https://en.wikipedia.org/wiki/Irving_S._Reed>
* <https://en.wikipedia.org/wiki/Gustave_Solomon>
* <https://en.wikipedia.org/wiki/Elwyn_Berlekamp>
* <https://en.wikipedia.org/wiki/James_Massey>
* <https://en.wikipedia.org/wiki/Lloyd_R._Welch>
* <https://en.wikipedia.org/wiki/Robert_Tienwen_Chien>
* <https://en.wikipedia.org/wiki/Dave_Forney>

