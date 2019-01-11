---
title : "Detailed Hash Function"
date: 2018-11-27T19:12:35+11:00
draft: false
---
# Hash Function

There will be three topics related to **HASH FUNCTION** in this article. Generally, **Hash Function** could be divided into two part, *Block cipher construct* and *dedicated hash function*. Our focus is on dedicated function, which contains three aspects.

1. MD4 Family
    1. MD5
    2. SHA-1
    3. SHA-3
2. SHA-3
3. Others

## Properties of Hash Function

## SHA-1

### Overview

The general idea for SHA-1 is to transfer the message with any length into 160 bits.

                                __________
                                |        |   (160 bits)
        x(input)  ------------> | SHA-1  |--------------> H(x)
                                |________|

### Internal Construction

SHA-1 uses the Merkle-Damard construction to implement HASH function(already broken).

Here we shows a more specific SHA-1 function diagram.

            x = (x_1,x_2,.....,x_n)
                    |
                    |
                ____|____
                |Padding|
                |_______|
                    |                      ______________________
                    | X_A(512 bits)        |                    |
                    |                      | H_A-1(160bits)     |
                ____|______________________|________            |
                \       Compression function      /             |
                 \_______________________________/              |
                           |    ^                               |
                           |    |_______________________________|
                           |
                        H(x) (160bits)

Further more, the technical detailed construction of compression function is :

             __________________
            | Message schedule |
            |__________________|                __________________
                    |                           | H_i-1(160 bits) |
                    |     W_0(32 bits)      ____|____             |
                    |--------------------->| Round 0 |            |
                    |                      |_________|            |
                    |                           |                 |
                    |                           |                 |
                    |     W_1(32 bits)      ____|____             |
                    |--------------------->| Round 1 |            |
                    |          .           |_________|            |
                    |          .                |                 |
                    |          .                |                 |
                    |                           |                 |
                    |     W_79(32 bits)     ____|_____            |
                    |--------------------->| Round 79|            |
                                           |_________|            |
                                                |                 |
                                                |                 |
                                            ____|_____            |
                                           |   EOR   |<-----------|
                                           |_________|

### SHA-1 Properties

1. Num of Rounds : 20*4 = 80 rounds
2. Num of Stages : 4
    Stage 1------> Round j = 0-19
    Stage 2------> Round j = 20-39
    Stage 3------> Round j = 40-59
    Stage 4------> Round j = 60-79
3. Input of each Round:
    1. 5*32 bits input(A,B,C,D,E)
    2. Message input W_j (32 bits)

                    A      B      C       D       E
                   _|______|______|_______|_______|_
            W_j-->|             Round j             |
                  |_________________________________|

### Remaining Problem when implementing

1. Internal algorithm of rounds
2. Construction of Message Schedule

<1> Round Function

             _________________________________________________
            |    A    |    B    |    C    |    D    |    E    |
            |_________|_________|_________|_________|_________|
             |   |       | |        |         | |       |
             |   |       | |        |         | |       |
             |   |       | _\_______|        /  |       |
             |   |       | | \ _____|______ /   |       |
             |   |       | |  |     f_t    |----|----->EOR
             |   |       | |  |____________|    |       |
             |   |       | |                    |       |
             |   |       | \                    |       |
             |  _|___    |  \                   |       |
             | |<<<5 |---|---\------------------|----->EOR
             | |_____|   |    \                 |       |
             |           |     \                |       |
             |         __|__    \               |       |
             |        |<<<30|    \              |      EOR<-------W_j
             |        |_____|     \             |       |
              \          |         \_________   |       |
               \         |                   |  |       |
                \        |                   |  |      EOR<-------k_t
                 \        \                  |  |       |
                  \        \                 |  |      /
                ___\________\________________|__|_____/
               |    \        \_______        |  |
               |     \               |       |  |_________
               |      \              |       |            |
             __|_______\_____________|_______|____________|___
            |    A    |    B    |    C    |    D    |    E    |
            |_________|_________|_________|_________|_________|

f_t(function) and k_t(constant) are constant given input according to the number of stage t.

<2>Message Schedule
As we mentioned round function before, there is a kind of input named w_j, which is constant number. Here we show the calculation process for each round(there will be 80 different values of w as there are 80 rounds in SHA-1).

             X_A
              |
              |--------512 bits
              |
              |---w_0
              |
              |---w_1
              |
              |    .
              .    .
              .    .
              .    .
              |____w_79

                    |<-       512 = 32*16        ->|
              x_i = x_i^0, x_i^1, x_i^2,.....,x_i^15
                      |      |      |           |
                      |      |      |           |
                     w_0    w_1    w_2   ...   w_15 .............w_79

            w_j = w_j-16 XOR x_j-14 XOR x_j-8 XOR x_j-3  (16 <= j <= 79)

## SHA-3

The algorithm of SHA-3 is much more complex than  SHA-1. Here we only record the high level of it and it's totally different from the algorithm of SHA-1. The core algorithm is Keccark.

The main thought of construction is kind of sponge construction.
    -absorbing phase: input x_i is read-in and processed.
    -squeezing phase: output is produced.

The parameter of sponge construction:
    -state:   b = 25*2^e,  e = 0, 1, 2, 3, 4, 5, 6(SHA-3).
    -number of round:    n_r = 12 +2l ------> 12 + 2*6 = 24

Here we show the table of sponge construction.
        ouput   |   b   |   r   |   c   |
        --------|-------|-------|-------|--
        224     | 1600  |  1152 |  448  |
        --------|-------|-------|-------|--
        256     | 1600  |  1088 |  512  |
        --------|-------|-------|-------|--
        384     | 1600  |  832  |  768  |
        --------|-------|-------|-------|--
        512     | 1600  |  576  |  1024 |
        --------|-------|-------|-------|--

The general idea of Kaccak algorithm is :

        m ---------> preprocessing --------> Inner keccak --------> y_2,y_1,y_0 = h(m)
        padding --->

# Reference

Finally, there are two ref videos introducing the algorithms of Hash Function perfectly. The firstly video introduced the SHA-1 while the second focused on SHA-3.

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/JIhZWgJA-9o/0.jpg)](http://www.youtube.com/watch?v=JIhZWgJA-9o)