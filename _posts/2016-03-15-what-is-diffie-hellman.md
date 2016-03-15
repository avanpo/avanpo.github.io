---
title: "What is Diffie-Hellman?"
categories: crypto key-exchange
---

<p class="preface">This article assumes knowledge of basic group theory, and basic computational complexity theory.</p>

For efficiency reasons, most encryption used on the internet and elsewhere is done using symmetric-key algorithms. A symmetric key algorithm uses one key for both encryption and decryption. This requires the sender and receiver to share knowledge of a secret key. For two parties who have never communicated securely before, this is a non-trivial problem. How does one establish a shared secret using insecure channels? This is where a key exchange algorithm like Diffie-Hellman comes in.

## Introduction

A key exchange algorithm is pretty self-explanatory. Given an insecure communication channel and no prior communication, two parties should be able to agree on some kind of shared secret using this algorithm, without an eavesdropper learning anything about this secret. For the rest of this article, we will assume the eavesdropper (attacker) is passive, and cannot actively interfere with the communication. Diffie-Hellman is unauthenticated on its own and does not prevent such active attacks.

The Diffie-Hellman key exchange algorithm is based on the "difficulty" of a certain mathematical problem, namely the computation of the discrete logarithm. No one has actually proven that the discrete logarithm can't be solved (in general) by an efficient classical algorithm. However, mathematicians and computer scientists alike have tried for many years to come with such an algorithm, and failed. Most therefore believe it doesn't exist. Cryptographers aren't the betting type, but mathematics is a topic they will bet on. The security of Diffie-Hellman is therefore based on the assumption that the discrete logarithm admits no efficient general solution.

#### Definition 1 (Discrete logarithm).

Let $$G$$ be any finite cyclic group of order $$n$$. Let $$g$$ be a generator of $$G$$, and let $$y \in G$$. The discrete logarithm of $$y$$ to the base $$g$$ is the unique integer $$k$$, $$0 \leq k \leq n - 1$$, such that $$y = g^k$$. We say $$k = \log_g y$$.

Given $$G$$, $$g$$, and $$y$$, the discrete logarithm problem is a matter of calculating $$k$$. For some groups, efficient algorithms are known. But others are considered intractable, and these are the ones exploited for cryptographic purposes.

## Diffie-Hellman key exchange

The original specification [1] used the multiplicative group of integers modulo a prime number, $$p$$. This group is typically denoted by $$G = (\mathbb{Z}/p\mathbb{Z})^{\times}$$. If $$p$$ is prime, the order of this group is $$p-1$$. This seems like a complex way to define a group, but it's simply the set of integers from 1 to $$p-1$$ inclusive, and integer multiplication modulo $$p$$ as the group operation. We choose the base $$g$$ to be a primitive root modulo $$p$$ (i.e. a generator of $$G$$).

Other groups are used as well, see for example elliptic curve Diffie-Hellman. But unless otherwise specified, Diffie-Hellman generally refers to the multiplicative group of integers modulo $$q$$. Given this group, the Diffie-Hellman key exchange algorithm is as follows. For convenience, private values are given in red, while public values (sent freely over the wire) are in blue.

- Alice and Bob agree on a prime $$\definecolor{red}{RGB}{255,0,0} \definecolor{blue}{RGB}{0,0,255} \textcolor{blue}{p}$$ (note that this uniquely defines $$G$$) and generator $$\textcolor{blue}{g}$$.
- Alice chooses a secret $$\textcolor{red}{a}$$, and sends Bob value $$\textcolor{blue}{A} = \textcolor{blue}{g}^{\textcolor{red}{a}} \mod \textcolor{blue}{p}$$.
- Bob chooses a secret $$\textcolor{red}{b}$$, and sends Alice value $$\textcolor{blue}{B} = \textcolor{blue}{g}^{\textcolor{red}{b}} \mod \textcolor{blue}{p}$$.
- Both compute shared secret $$\textcolor{red}{s} = \textcolor{blue}{g}^{\textcolor{red}{ab}} \mod \textcolor{blue}{p}$$.

Modular exponention is cheap computationally, so it is easy for Alice to calculate $$A$$. Similarly, she can efficiently calculate $$s = B^a \mod p$$. Comparatively, an eavesdropper does not know $$a$$ or $$b$$, and therefore cannot easily find the secret $$s$$. While the security of Diffie-Hellman has never been proven equivalent to the discrete logarithm problem, computing the discrete log remains the best known attack. Therefore, barring any flaws in the implementation, an attacker's best bet is to calculate $$a$$ or $$b$$ from $$A$$ or $$B$$.

## Finding the discrete logarithm

Previously it was mentioned that no efficient algorithms to compute the discrete log exist. What exactly is meant by efficient?

A simple algorithm to find the discrete log is a simple brute force search. Given a base $$g$$ and a $$y$$, try different values of $$k$$ until $$g^k = y$$. The running time of this algorithm is $$O(n)$$, or linear in the size of the order of $$G$$. This is clear by noting that we will potentially need to try every $$k \in G$$, to find $$y$$. Recall that the order of $$G$$ is $$p-1$$. If we increase the size of $$p$$ by one bit, we've effectively doubled the running time of this algorithm. Therefore, it is exponential in the number of digits of $$p$$. It is evident that increased sizes of $$p$$ very quickly make it infeasible to compute the discrete log, using this algorithm.

Generally, an "efficient" algorithm refers to a polynomial time algorithm. In this case, the algorithm that doesn't appear to exist is a polynomial-time algorithm in the number of bits of $$p$$. While this remains the case, there are several algorithms that are significantly faster than the naive brute force algorithm above.

The most efficient algorithm known for discrete log computation is the Number Field Sieve (NFS). You may recognize the name, a variant (same name) is used to factor large integers. The specific details of this algorithm are complex, and outside the scope of this article. Many parameters are used, which can shift computational burdens between the four different steps. When properly tuned parameters are used, the running time of the algorithm is $$O(\exp((1.923 + o(1))(\log p)^{\frac{1}{3}}(\log \log p)^{\frac{2}{3}})$$. Most of the algorithm (the first three steps, including the most expensive two) only depends on the group, and not the specific discrete log being calculated.

Another notable algorithm is the Pohlig-Hellman algorithm, designed especially for groups whose order is a smooth integer. A smooth integer is an integer which factors completely into small primes numbers $$q_i$$. The running time of this algorithm is $$O(\sum_i \log p + \sqrt{q_i})$$. Clearly, this becomes very fast if the number is indeed smooth, but reduces to $$O(\sqrt{p})$$ if $$p-1$$ has one large prime factor.

## Security

As previously stated, the security of Diffie-Hellman has never been proved equivalent to the discrete logarithm problem. What does this mean? The eavesdropper has access to $$g^a$$ and $$g^b$$, and is looking to find $$g^{ab}$$ (called the Diffie-Hellman problem). This is different from the discrete logarithm problem. It is trivial to show that solving the discrete logarithm gives you a solution to the Diffie-Hellman problem, but the reverse has proven more difficult.

Second, whether or not the discrete logarithm can be computed in polynomial-time on a classical computer is still an open problem. Whether or not this is possible on a quantum computer has already been solved. It is, thanks to [Shor's algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm).

In any case, the Diffie-Hellman protocol has been around for 40 years, seen widespread use, and still has not been broken. It is widely believed to be secure if the $$G$$ and $$g$$ are chosen correctly. To avoid use of the Pohlig-Hellman algorithm, the prime $$p$$ characterizing $$G$$ must be a safe prime. In other words, $$p-1$$ must not be a smooth integer, i.e. contain a large prime factor. This is most often done by setting $$p = 2q + 1$$, where $$q$$ is another prime, called a Sophie-Germain prime. This renders Pohlig-Hellman inefficient and leaves NFS as the best attack.

Despite this belief of security, Diffie-Hellman saw a high-profile attack in 2015, called Logjam [2]. The attack exploited a flaw in TLS to downgrade a Diffie-Hellman connection to export-grade strength of 512 bits. The attack affected a limited number of servers -- only those that still supported export-grade crypto. This amounted to approximately 4-8% of browser-trusted sites. The authors also successfully attacked misconfigured servers, using unsafe primes. While this attack was dangerous in its own right, the methods used revealed an often glossed-over fact of the discrete logarithm over the multiplicative group of integers modulo $$q$$.

NFS supports a large amount of precomputation, if the group is known. The authors found that the vast majority of the internet used only one or two groups. Thus the cost of an attack could be amortized to a huge degree across targets. This is the technique used in Logjam, to break the two 512-bit groups used by 93% of servers supporting this export-grade crypto. However, the authors also extrapolated to 1024-bit groups, and estimated that breaking such a group would be within the reach of an adversary with state-level resources. In particular, comparisons were drawn to documents leaked by Edward Snowden which showed the NSA being able to passively decrypt huge numbers of VPN connections.

In short, Diffie-Hellman as a protocol is most likely secure, when used with primes larger or equal to 2048 bits. That said, elliptic curve Diffie-Hellman uses smaller parameters and isn't vulnerable to the same precomputational woes as regular DH. For these reasons, Diffie-Hellman has fallen largely out of favor, and the elliptic curve variant is recommended instead.

## References

[1] Diffie, Whitfield, and Martin E. Hellman. "New directions in cryptography." *Information Theory, IEEE Transactions on* 22.6 (1976): 644-654.

[2] Adrian, David, et al. "Imperfect forward secrecy: How Diffie-Hellman fails in practice." *Proceedings of the 22nd ACM SIGSAC Conference on Computer and Communications Security*. ACM, 2015.
