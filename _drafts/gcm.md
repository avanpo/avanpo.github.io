# Galois/Counter Mode

AES in combination with GCM has become the de facto standard for symmetric cryptography, and for good reason. It is an unpatented authenticated encryption algorithm, and does away with a lot of the pitfalls that have led to devastating breaks for other modes of operations.

It's not that other modes aren't safe, because they can be. But modern crypto is less about increasing the security of the underlying primitives, and more about ensuring that the average developer doesn't use them incorrectly.

But how does it work? What does authentication mean, in the context of a symmetric encryption algorithm?
