---
title: "J-PAKE for .NET"
categories: crypto
---

A J-PAKE implementation is now available for .NET and other C# projects.

J-PAKE is short for Password Authenticated Key Exchange by Juggling. Using a PAKE protocol, two parties are able to establish private and authenticated communication by relying on a weak shared password instead of public key infrastructure. Diffie-Hellman is another password key exchange protocol, but lacks the authentication.

Bouncy Castle has had a Java implementation since 2013, but its C# project is significantly less well-maintained. I ported the implementation to C#, and it's been merged into the C# Bouncy Castle distribution.

### A word of warning

It should be mentioned that J-PAKE is still maturing. A formal security proof only came along in 2015 [1]. As a result, it's questionable whether the implementations are sound.

There doesn't appear to be a standard RFC, only an expired draft [2]. Interop is a concern, and indeed there appear to be some inconsistencies between the OpenSSL and Bouncy Castle implementations.

1. M. Abdalla, F.Benhamouda, P. MacKenzie [Security of the J-PAKE Password-Authenticated Key Exchange Protocol](http://www.normalesup.org/~fbenhamo/files/publications/SP_AbdBenMac15.pdf)
2. F. Hao [J-PAKE: Password Authenticated Key Exchange by Juggling](https://tools.ietf.org/html/draft-hao-jpake-01)
