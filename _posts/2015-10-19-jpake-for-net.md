---
title: "J-PAKE for .NET"
categories: crypto
comments: true
---

A C# J-PAKE implementation is now available for .NET and other C# implementations.

J-PAKE is short for Password Authenticated Key Exchange by Juggling. Using a PAKE protocol, two parties are able to establish private and authenticated communication by relying on a (possibly low-entropy) shared password instead of public key infrastructure.

Bouncy Castle has had a Java implementation since 2013, but its C\# project is significantly less well-maintained. I ported the implementation to C\#, and it's been merged into the C\# Bouncy Castle distribution [1].

### Alternatives

Diffie-Hellman is another key exchange protocol, but it lacks authentication. Constructing an authentication scheme is possible, but difficult to do correctly (like all crypto). A second alternative is SRP [2], another PAKE protocol. Its adoptation was slowed by patent encumbrance, but it appears this is no longer a problem.

### A Word of Warning

It should be mentioned that J-PAKE is still maturing. A formal security proof only came along in 2015 [3]. The implementations are only a few years old, and aren't widely used. As a result, it's questionable whether the implementations are secure.

A standard RFC is lacking, only an expired draft exists [4]. Interop is a concern, and indeed there appear to be some inconsistencies between the OpenSSL and Bouncy Castle implementations.

1. [The Legion of the Bouncy Castle](https://www.bouncycastle.org/csharp/)
2. [The Standford SRP Homepage](http://srp.stanford.edu/)
3. M. Abdalla, F.Benhamouda, P. MacKenzie [Security of the J-PAKE Password-Authenticated Key Exchange Protocol](http://www.normalesup.org/~fbenhamo/files/publications/SP_AbdBenMac15.pdf)
4. F. Hao [J-PAKE: Password Authenticated Key Exchange by Juggling](https://tools.ietf.org/html/draft-hao-jpake-01)
