---
title: "What is J-PAKE?"
categories: crypto key-exchange
---

<p class="preface">This article assumes knowledge of basic group theory.</p>

A bit over a year ago I contributed to the C\# Bouncy Castle crypto API, by porting a J-PAKE implementation already present in the Java project to C\#.

At the time I didn't really understand how J-PAKE works. So why did I port it, then? That's a good question. I shouldn't have. But my employer at the time wanted me to, so I did it anyway.

Now that I know a bit more about crypto, let's take a look at how it works.

## So how does it work?

The acronym J-PAKE stands for **Password Authenticated Key Exchange by Juggling**. As the name implies, it is a password authenticated key agreement protocol. The key agreement part is based on Diffie-Hellman, so it offers forward secrecy. Since the password is only used for authentication, it does not need a large amount of entropy.

To understand J-PAKE, it helps to understand how Diffie-Hellman works. I wrote about that here, but any article will do. Come back when you've grokked DH, and we'll continue.
