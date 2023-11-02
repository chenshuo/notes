# Compact Disc

https://web.archive.org/web/20181122182956/http://www.cs.tut.fi/~ypsilon/80545/CD.html

## Table of Contents

------------------------------------------------------------------------

## <span id="HDR0">LP versus CD</span>

-   LP
    -   LP stores its information as an analog groove.
    -   Variations in its side-to-side amplitude and depth represents the original audio data.
    -   Left and right channels are stored on either side of the groove walls.
    -   Mechanical movements of the stylus are converted into an electrical signal.
-   Problems with LP
    -   Handle with care: Groove damage is audible.
    -   SNR 60 dB, channel separation 30 dB, frequency response not flat.
    -   The mechanical transport introduces audible artifacts such as
        wow and flutter.
    -   The stylus requires periodic replacement.
    -   Electrical circuitry is prone to aging.
    -   Phase shifts introduced by analog circuits.
    -   Signal equalization needed.
-   CD
    -   Information is stored digitally.
    -   The length of its data pits represents a series of 1s and 0s.
    -   Both audio channels are stored along the same pit track.
    -   Data is read using laser beam.
    -   Information density about 100 times greater than in LP.
    -   CD player can correct disc errors.
-   Benefits of CD
    -   Robust
    -   No degradation from repeated playings because data is read by
        the laser beam.
    -   Error correction
    -   Transport's preformance does not affect the quality of audio
        reproduction.
    -   Digital circuitry more immune to aging and temperature problems
    -   Data conversion is independent of variations in disc rotational
        speed, hence wow and flutter are neglible.
    -   SNR over 90 dB.
    -   Subcode for display, control and user information

## <span id="HDR1">Compact Disc digital format</span>

-   Sampling frequency
    -   44.1 kHz =\> 10 % margin with respect to the Nyquist frequency
        (audible frequencies below 20 kHz)
-   Quantization
    -   16-bit linear =\> theoretical SNR about 98 dB (for sinusoidal signal with
        maximum allowed amplitude)
    -   2's complement
-   Signal format
    -   audio bit rate 1.41 Mbit/s (44.1 kHz \* 16 bits \* 2 channels)
    -   Cross Interleave Reed-Solomon Code ([CIRC](https://en.wikipedia.org/wiki/Cross-interleaved_Reed%E2%80%93Solomon_coding))
    -   total data rate (CIRC, sync, subcode) 2.034 Mbit/s
    -   modulation method 8 to 14 modulation ([EFM](https://en.wikipedia.org/wiki/Eight-to-fourteen_modulation))
        8-bit data are converted to 14+3 channel bits after modulation
    -   channel bit rate 2.034 \* 17/8 = 4.3218 Mbit/s
-   Playing time
    -   max. 74.7 min
-   Disc spesifications
    -   diameter 120 mm
    -   thickness 1.2 mm
    -   track pitch 1.6 µm
    -   one side medium
    -   disc rotates clockwise
    -   signal is recorded from inside to outside
    -   constant linear velocity (CLV) recording maximizes recording
        density =\> the speed of revolution of the disc is not constant;
        it gradually decreases from 500 to 200 r/min
    -   pit is about 0.5 µm wide
    -   each pit edge is 1 and all areas in between, whether inside or
        outside a pit, are 0s

## <span id="HDR2">Error Correction</span>

-   A typical error rate of a CD system is 10^-5, which means that a data
    error occurs 20 times per second.
-   About 200 errors/s can be corrected.

Sources of errors:

-   dust
-   scratches
-   fingerprints
-   pit asymmetry
-   bubbles or defeacts in substrate
-   coating defects
-   dropouts

Objectives for error correction

-   powerful error correction capability for random and burst errors
-   reliable error detection in case of an uncorrectable error
-   low redundancy =\> CIRC satisfies these criteria

## <span id="HDR3">Cross Interleave Reed-Solomon Code (CIRC)</span>

-   two correction codes for additional correcting capability
    -   C2 can effectively correct burst errors.
    -   C1 can correct random errors and detect burst errors.
-   Three interleaving stages to encode data before it is placed on a
    disc.
-   Parity checking to correct random errors
-   Cross interleaving to permit parity to correct burst errors.

1\. Input stage:

-   12 words (16-bit, 6 words per channel) of data per input frame
    divided into 24 symbols of 8 bits

2\. C2 Reed-Solomon code:

-   24 symbols of data are enclosed into a (28,24) R-S code
-   4 parity symbols (Q) are used for error correction
    ( (*n*,*k*) linear block code: A group of *k* symbols (data) is
    encoded to a longer word of *n* symbols (the code word). )

3\. Cross interleaving:

-   to guard again the burst errors
-   separete error correction codes
-   one code can check the accuracy of another
-   error correction is enhanced

4\. C1 Reed-Solomon code:

-   cross-interleaved 28 symbols of the C2 code are encoded again into a (32,28) R-S code
-   4 parity symbols (P) are used for error correction
-   effective for random-error correction and burst-error detection

5\. Output stage:

-   half of the code word is subject to a 1-symbol delay to avoid
    2-symbol error at the boundary of symbols

## <span id="HDR4">Performance of CIRC</span>

-   Both R-S coders (C1 and C2) have four parities, and their minimum
    distance is 5
    (minimum distance: the number of bits that on code word must change
    to become another code word)
-   If error location is not known, up to two symbols can be
    corrected.
-   If the errors exceed the correction limit, they are concealed by
    interpolation.
-   Since even-numbered sampled data and odd-numbered sampled data are
    interleaved as much as possible, CIRC can conceal long burst errors
    by simple linear interpolation.
-   Max. completely correctable burst length is about 4000 data bits
    (2.5 mm track length).
-   Max. interpolatable burst length in the worst case is about 4000
    data bits (7.7 mm).
-   Sample interpolation rate is one sample every 10 hours at BER (Bit
    Error Rate) = 10^-4 and 1000 samples at BER = 10^-3.
-   Undetectable error samples (clicks) less than one every 750 hours at
    BER = 10^-3 and negligible BER = 10^-4.

## <span id="HDR5">Subcode</span>

-   Following CIRC encoding, an 8-bit subcode symbol is added to each frame.
-   Eigth subcode bits are designated as P, Q, R, S, T, U, V, W.
-   The CD player collects subcode symbols from 98 consecutive frames to
    form a subcode block, with eight 98-bit words.
-   Only P and Q bits are used in audio CD's.

## <span id="HDR6">Modulation</span>

-   CIRC coded data cannot be directly recorded on the disc =\> EFM modulation is used.
-   EFM modulation after the audio, parity, and subcode data are assembled.
-   Blocks of 8 data bits are converted into blocks of 14 bits, known as
    channel bits, using a ROM dictionary which assigns an unambiguous
    word of 14 bits to each 8-bit word.
-   14-bit words are selected that have low number of transitions between 0 and 1
    -   those combinations which contain at least two but no more than
        then consecutive zeros (267 patterns satisfy criteria).
        =\> greater data density
-   A kind of error correction, because more unique patterns can be
    selected than if 8-bit words were directly recorded.

Merging bits

-   Blocks of 14 bits are linked by 3 merging bits.
-   Two merging bits (0s) are required to prevent the possibility of
    successive 1s between serial words.
-   Additional mergin bit (1 or 0) is to aid in clock synchronization
    and to supress the signal low frequency component.
-   Merging bits are discarded during demodulation.

Pit length

-   Signal exists as a non-return-to-zero signal (NRZ), so signal level
    high at 1 and low at 0.
-   After EFM, signal is converted to non-return-to-zero-inverted (NRZI) form.
-   NRZI gives fewer transitions and simplifies the pit structure on the disc.
-   Pits and lands are at least 3 channel bits and no more that 11 bits long
    =\> 3T pit is thus highest frequency signal (720 kHz) while 11T pit
    is the lowest (196 kHz).

Frame assembly

-   Individual frames in resulting EFM bit stream must be delineated
    =\> synchronization pattern (24 bits) is added prior each frame.
-   Sync pattern is needed to make the bit stream self-clocking.
-   Sync word is unique.

<span id="ENDFILE"></span>


