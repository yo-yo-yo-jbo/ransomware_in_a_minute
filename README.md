# Coding a ransomware in a minute
I'm taking a little "side-quest" from my macOS introduction series to talk about ransomware.  
Sometimes I do defensive security work - in this case, I'd like to explain why stopping ransomware is so challenging, as well as show how to create a ransomware in a minute.  
This will also give us a nice opportunity to talk a bit about encryption.

## What is ransomware
Ransomware is a type of malware that has the sole purpose of encrypting files that are important to the target human.  
After encryption, the ransomware "reveals" itself by announcing that files have been encrypted, demanging "ransom" for releasing them.  
Basically, ransomware is just software that holds your *data* hostage.  
While varying in sophistication, the "ransoming" part is the interesting part, which is what I'd like to touch.

## Encryption 101
When we talk about encryption, we basically talk about two classes:
- Symmetric encryption: encryption key and decryption key are the same.
- Asymmetric encryption: encryption key and decryption key are different.

While it sounds minor, this has huge implications. It's not an exaggeration to say that much of our modern society is built on these concepts!  
For example, our bank transactions, online shopping, emails and everything else is (I hope!) protected by a network protocol called SSL\TLS.  
That protocol uses both symmetric and asymmetric encryption to ensure several properties that ensure your privacy is safe.

Symmetric encryption has been around for hundreds of years (really!). When we talk about symmetric encryption, we generally have two categories:
- Block ciphers: the cryptosystem works on *blocks* of data, e.g. every encrypts every 16 bytes. This means that the plaintext has to be padded (which might be a source of vulnerabilities I might discuss in a future blogpost). In a naive implementation, block ciphers output the same encrypted block for two identical input blocks.
- Stream ciphers: the cryptosystem works on each byte of data. You can think of stream ciphers as a blackbox that gets a key and spits out an infinite stream of pseaudo-random bytes. Those bytes are `XOR`'ed with the plaintext to create the encrypted data. This is also why stream ciphers are different from block ciphers with 1 byte - two identical plaintext bytes produce different encrypted output bytes. Think why that property is critical for the safety of the cryptosystem!

The most commonly used symmetric encryption nowadays is [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), which in its essence - a Block Cipher.  
I've also seen ransomware use [RC4](https://en.wikipedia.org/wiki/RC4) as well (which is a Stream cipher).  
Because of the property that identical plaintext blocks produce identical encrypted output blocks (and obviously vice-versa), all Block ciphers can use a [Mode of Operation](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation). The Mode of Operation prevents that from happening by using additional information from each block as input to another block, and a new known initial block of information called an `Initialization Vector (IV)`.  
There are many Modes of Operation; one of the most prevalent one is `CBC`. CBC performs a `XOR` operation between the output of the previous block and the plaintext input of the next block. Here's some ASCII art that shows how that works, for instance, with `AES`:

```
Plaintext:  P1            P2            P3    ...           Pn
            |             |             |                   |
            |             |             |                   |
            |             |             |                   |
IV ------> XOR      |--> XOR      |--> XOR    ...     |--> XOR
            |       |     |       |     |             |     |
            |       |     |       |     |             |     |
           AES      |    AES      |    AES            |    AES
            |-------|     |-------|     |---- ... ----|     |
            |             |             |                   |
            V             V             V                   V
            E1            E2            E3    ...           En
```

If we mark `IV` as `E0`, we can make a nice formula: `E[n] = AES-encrypt(P[n] XOR E[n-1])`.






