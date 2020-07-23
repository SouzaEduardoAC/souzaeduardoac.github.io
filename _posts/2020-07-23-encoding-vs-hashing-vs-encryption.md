---
layout: post
title: Encoding vs Hashing vs Encryption
author: Eduardo Souza
date: '2020-07-23 20:21:23 -0300'
category:
        - security
summary: In this post, we will see the difference in each technique
thumbnail: /assets/img/posts/encryption.jpg
---

**Encoding** is a technique to transform data from one form to another that suits best the system that processes this data.
>  * Can also be used to compress data in transit or storage
>  * Should never be used to keep data confidential

**Hashing** is a one-way transformation of arbirary data into a value of the fixed lenght.
>  * Cryptographic
>  * Noncryptographic -> should never be used for security related

**SHA: Chryptographic hashing algorithms** should have the following laws:
 
>  * Preimage resistance, it's impossible to derive the original message
>  * Unpredictability -> the output should be impredictable

Also having quick computation time, hashing is used to ensure the integrity of data with _digital signatures_ and _TLS certificates_.

**Encryption** is the transformation of data into a format that doesn't allow to derive the initial meaning from it even if an attacker gets access to it and should be used if data confidentiality is the goal, **it needs to be decrypted using the _cryptographic key_**.
There are two types of algorithms to be used: symmetric and asymmetric, symmetric have better performance than asymmetric.
> * Symmetric use the same key for encrypt and decrypt
>   * widely used for encryption of arbitrary data
>   * can be used by two endpoints who share the same secret

> * Asymmetric use a pair of public and private keys for encrypt/decrypt, encrypting with one key and decrypting with the other. 
>   * used for secret key distribution and digital signatures
>   * can be used by an endpoint with multiple clients who trusts the endpoint but don't want anyone listening in
>   * used for implicit authentication, if anyone can provide a token that the application can decrypt with the public key, then it must have been encrypted with the private key, which can only belong to trusted endpoints (_Kerberos for example_)