---
title: "Diffie-Hellman"
categories: crypto key-exchange
---

<p class="preface">This article assumes knowledge of basic group theory, and basic computational complexity theory.</p>

For efficiency reasons, most encryption used on the internet and elsewhere is done using symmetric-key algorithms. A symmetric key algorithm uses one key for both encryption and decryption. This requires the sender and receiver to share knowledge of a secret key. For two parties who have never communicated securely before, this is a non-trivial problem. How does one establish a shared secret using insecure channels? This is where a key exchange algorithm comes in.

A key exchange algorithm is pretty self-explanatory. Given an insecure communication channel and no prior communication, two parties should be able to agree on some kind of shared secret using this algorithm, without an eavesdropper learning anything about this secret. We will assume the eavesdropper is passive, and cannot actively interfere with the communication. Diffie-Hellman is unauthenticated on its own and does not prevent such active attacks.

The Diffie-Hellman key exchange algorithm is based on the "difficulty" of a certain mathematical problem, namely the computation of the discrete logarithm. It should be noted that no one has proved that the discrete logarithm can't be solved (in general) by an efficient classical algorithm. However, mathematicians and computer scientists alike have tried for many years to come with such an algorithm, and failed. So most assume the problem is computationally intractable. Cryptographers aren't the betting type, but this is one conjecture they will bet on. The security of Diffie-Hellman is therefore based on the assumption that the discrete logarithm admits no efficient solution.

#### Definition 1 (Discrete logarithm).

Let $$G$$ be any finite cyclic group of order $$n$$. Let $$g$$ be a generator of $$G$$, and let $$y \in G$$. The discrete logarithm of $$y$$ to the base $$g$$ is the unique integer $$k$$, $$0 \leq k \leq n - 1$$, such that $$y = g^k$$. We say $$k = \log_g y$$.

Given $$G$$, $$g$$, and $$y$$, the discrete logarithm problem is a matter of calculating $$k$$. For some groups, efficient algorithms are known. But others are considered intractable, and these are the ones exploited for cryptographic purposes.

## Diffie-Hellman key exchange

The original specification [1] used the multiplicative group of integers modulo a prime number, $$p$$. This group is typically denoted by $$G = (\mathbb{Z}/p\mathbb{Z})^{\times}$$. If $$p$$ is prime, the order of this group is $$p-1$$. This seems like a complex way to define a group, but it's simply the set of integers from 1 to $$p-1$$ inclusive, and integer multiplication modulo $$p$$ as the group operation. We choose the base $$g$$ to be a primitive root modulo $$p$$ (i.e. a generator $$G$$).

Other groups are used as well, see for example elliptic curve Diffie-Hellman. But unless otherwise specified, Diffie-Hellman generally refers to the multiplicative group of integers modulo a prime. Given this group, the Diffie-Hellman key exchange algorithm is as follows. For convenience, private values are given in red, while public values (sent freely over the wire) are in blue.

* Alice and Bob agree on a prime $$\textcolor{blue}{p}$$ and generator $$\textcolor{blue}{g}$$.
* Alice chooses a secret $$\textcolor{red}{a}$$, and sends Bob value $$\textcolor{blue}{A} = \textcolor{blue}{g}^{\textcolor{red}{a}} \mod \textcolor{blue}{p}$$.
* Bob chooses a secret $$\textcolor{red}{b}$$, and sends Alice value $$\textcolor{blue}{B} = \textcolor{blue}{g}^{\textcolor{red}{b}} \mod \textcolor{blue}{p}$$.
* Both compute shared secret $$\textcolor{red}{s} = \textcolor{blue}{g}^{\textcolor{red}{ab}} \mod \textcolor{blue}{p}$$.

Modular exponention is cheap computationally, so it is easy for Alice to calculate $$A$$. Similarly, she can efficiently calculate $$s = B^a \mod p$$. Comparatively, an eavesdropper does not know $$a$$ or $$b$$, and therefore cannot easily find the secret $$s$$. While the security of Diffie-Hellman has never been proven equivalent to the discrete logarithm problem, computing the discrete log remains the best known attack. Therefore, barring any flaws in the implementation, an attacker's best bet is to calculate $$a$$ or $$b$$.

## Finding the discrete logarithm

Previously it was mentioned that no efficient algorithms to compute the discrete log exist. What exactly is meant by efficient?

A simple algorithm to find the discrete log is simply a brute force search. Given a base $$g$$ and a $$y$$, try different values of $$k$$ until $$g^k = y$$. The running time of this algorithm is linear in the size of the order of $$G$$. Therefore, it is exponential in the number of digits of this number. Recall that the order of $$G$$ is $$p-1$$. If we increase the size of $$p$$ by one bit, we've effectively doubled the running time of this algorithm. It is evident that it does not require very large $$p$$ to produce a group that makes it computationally infeasible to find the discrete log, regardless of the other parameters used.

Of course, this is a very simple algorithm, and there are faster algorithms available to compute the discrete log.

## References

[1] Diffie, Whitfield, and Martin E. Hellman. "New directions in cryptography." *Information Theory, IEEE Transactions on* 22.6 (1976): 644-654.
