---
title: "What is J-PAKE?"
categories: crypto key-exchange
---

<p class="preface">This article assumes knowledge of basic group theory.</p>

A bit over a year ago I contributed to the Bouncy Castle Crypto API, by porting a J-PAKE implementation already present in the Java project to C\#.

At the time I didn't really understand how J-PAKE works. So why did I port it, then? That's a good question. I shouldn't have. But my employer at the time needed me to, so I did it anyway.

Now that I know a bit more about crypto, let's take a look at how it works.

## J-PAKE

The acronym J-PAKE stands for **Password Authenticated Key Exchange by Juggling**. A PAKE protocol allows two or more parties to establish keys based on a shared secret (the password), while protecting from passive and active attackers. Since passwords are often low entropy, ideally the protocol will restrict an adversary's ability to perform a brute force attack.

In J-PAKE, the key agreement part is based on Diffie-Hellman, so it offers forward secrecy. The password is only used for authentication, in a way that prevents offline dictionary attacks and limits online attacks to one guess per execution of the protocol.

To understand J-PAKE, it helps to understand how Diffie-Hellman works. I wrote about that here, but any article will do. Come back when you've grokked DH, and we'll continue.

### Zero knowledge proofs


