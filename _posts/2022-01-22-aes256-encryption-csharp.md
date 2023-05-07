---
layout: post
title: AES256 Encryption in C#
date: 2022-01-22
author: Jason M Penniman
excerpt: Every few years, I find myself having to write an AES256 encryption routine for a project. I always end up having to lookup the specifics, so I thought I would write it down this time.
tags:
- dotnet
- C#
- encryption
- cryptography
---

Every few years, I find myself having to write an AES256 encryption routine for a project. I always end up having to lookup the specifics, so I thought I would write it down this time.

## What is AES256

AES--Advanced Encryption Standard--is a symmetrical block cipher standard. Symmetrical meaning it uses the same key to both encrypt and decrypt, as opposed to Asymmetrical (eg. TLS) which uses a public/private key pair--a different key to encrypt than to decrypt. Block cipher meaning the encryption is performed by chunking the plain text into blocks and encrypting each block separately. In the case of AES, each block is 128 bits in length. Chunks smaller than 128 bits are padded to create a 128 bit block.

At the time of this writing, bank-grade and government standard uses a 256 bit key (AES256) and a CBC--Cipher Block Chaining--mode. With CBC, each plain text block is XOR with the previous encrypted block before being encrypted itself. This requires an initialization vector--a unique, randomly generated block to XOR the first block with.

## Setting this up in C#

In C#, we use the Aes class to create encryptors and decryptors that use an AES algorithm.

``` csharp
var aes = Aes.Create();

aes.Mode = CipherMode.CBC; // NIST and NSA recommended cipher mode
aes.Padding = PaddingMode.PKCS7; // NIST and NSA recommended padding mode
aes.KeySize = 256; // AES256
aes.BlockSize = 128; // blocks must be 128 bits

// Consumer defined symmetrical key
aes.Key = _key;
```

## Initialization Vector

The initialization vector is unique for every encryption operation, even for the same plain value. This ensures that two sets of encrypted bytes for the same text are different so patterns can't be spotted. However, the same initialization vector that was used to encrypt the value must be used to decrypt it. To handle this, a typical practice is to prepend the initialization vector to the encrypted bytes and extract them when we perform the decryption. The initialization vector isn't something we need to keep secret. Our encryption key is piece we must keep secret and guard with our lives.

## Key Generation

We can create a key with the openssl cli and load it at run time. NEVER NEVER NEVER NEVER NEVER hardcode the key in the source code or store it in source control. It should be a deployment time artifact.

``` bash
openssl rand 32 > aes.key
```

Here's the full class. Error handling has been removed for clarity. This approach uses a CryptoStream which takes care of the blocking for us.

``` csharp
using System;
using System.IO;
using System.Linq;
using System.Security.Cryptography;

public class Crypto
{
    readonly byte[] _key;

    public Crypto(byte[] key)
    {
        if (key.Length != 32)
            throw new ArgumentException("Must be a 256bit (32 byte) key.");
            
        _key = key;
    }

    Aes CreateAes()
    {
        var aes = Aes.Create();

        aes.Mode = CipherMode.CBC;
        aes.Padding = PaddingMode.PKCS7;
        aes.KeySize = 256; // AES256
        aes.BlockSize = 128;

        // Consumer defined symmetrical key
        aes.Key = _key;

        return aes;
    }

    public byte[] Encrypt(string plainText)
    {
        byte[] encryptedBytesWithIv = null;

        using (var aes = CreateAes())
        {
            // Random, unigue initialization vector every time
            aes.GenerateIV();

            using (var encryptor = aes.CreateEncryptor())
            {
                using (var ms = new MemoryStream())
                {
                    using (var cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))
                    {
                        using (var wtr = new StreamWriter(cs))
                        {
                            wtr.Write(plainText);
                        }

                        var encyptedBytes = ms.ToArray();

                        // Since the IV in unique for every encryption attempt,
                        //  we'll store it as the first 16 bytes of the encrypted byte array.
                        encryptedBytesWithIv = aes.IV.Concat(encyptedBytes).ToArray();
                    }
                }
            }
        }

        return encryptedBytesWithIv;
    }

    public string Decrypt(byte[] encryptedBytes)
    {
        string plainText = null;

        using (var aes = CreateAes())
        {
            // During encryption, we stored the unique IV in the first 16 bytes
            aes.IV = encryptedBytes.Take(16).ToArray();

            // The actual value to decrypt is the rest of the bytes after the stored IV.
            var encryptedValue = encryptedBytes.Skip(16).ToArray();

            using (var decryptor = aes.CreateDecryptor())
            {
                using (var ms = new MemoryStream(encryptedValue))
                {
                    using (var cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Read))
                    {
                        using (var rdr = new StreamReader(cs))
                        {
                            plainText = rdr.ReadToEnd();
                        }
                    }
                }
            }
        }

        return plainText;
    }
}
```

Cheers!

PS. Don't use AES for storing encrypted passwords. Passwords should be encrypted using a one way hashing algorithm such as SHA256. One way meaning they cannot be decrypted. When a user supplies a username and password to your application, encrypt the supplied value and compare the two hashes.
