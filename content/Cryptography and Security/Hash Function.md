---
title : "Detailed Hash Function"
date: 2018-11-27T19:12:35+11:00
draft: false
---
### Hash Function

There will be three topics related to **HASH FUNCTION** in this article. Generally, **Hash Function** could be divided into two part, *Block cipher construct* and *dedicated hash function*. Our focus is on dedicated function, which contains three aspects.

1. MD4 Family
    1. MD5
    2. SHA-1
    3. SHA-3
2. SHA-3
3. Others

#### Properties of Hash Function

#### SHA-1

##### Overview

The general idea for SHA-1 is to transfer the message with any length into 160 bits.

                          --------     (160 bits)
 x(input)  ------------> |  SHA-1 | --------------> H(x)
                          --------

##### Internal Construction

SHA-1 uses the Merkle-Damard construction to implement HASH function(already broken).

Here we shows a more specific SHA-1 function diagram. 
            x = (x_1,x_2,.....,x_n)
                    |
                    |
                _________
                |Padding|
                |_______|
                    |                      ______________________
                    | X_A(512 bits)        |                    |
                    |                      | H_A-1(160bits)     |
                ____|______________________|________            |
                \       Compression function      /             |
                 \\_______________________________/              |
                           |    |________________________________|
                           |
                           |
                        H(x) (160bits)
##### SHA-1 Properties

##### Remaining Problem when implementing

***Insides of Rounds***

***Message Schedule***

#### SHA-3



#### Reference
Finally, there are two ref videos introducing the algorithms of Hash Function perfectly. The firstly video introduced the SHA-1 while the second focused on SHA-3.

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/JIhZWgJA-9o/0.jpg)](http://www.youtube.com/watch?v=JIhZWgJA-9o)