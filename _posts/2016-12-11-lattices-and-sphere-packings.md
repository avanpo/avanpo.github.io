---
title: "Lattices and sphere packings"
categories: lattices sphere-packing
---

<p class="preface">This article assumes basic knowledge of linear algebra and group theory.</p>

What is a lattice? Good question. First, we'll need to define some concepts.

## Definitions

When we think of the word *lattice*, we most likely envision some kind of spacial structure. The faces of a crystal, or the metal beams of a bridge that form a repeating pattern of shapes. Using mathematics, we can distill this structure down to a periodic arrangement of points in some $$n$$-dimensional space.

There are many formal ways to define a lattice, some many pages long. Let's restrict ourselves to Euclidean vector spaces, and use group theory to simplify our definition. We start with the definition of a discrete subset.

#### Definition 1 (Discrete subset).
A subset $$S$$ of $$\mathbb{R}^n$$ is discrete if for each $$\mathbf{x} \in S$$ there exists a positive real number $$\epsilon$$ such that the only vector $$\mathbf{y} \in S$$ with $$\|\mathbf{x} - \mathbf{y}\| < \epsilon$$ is given by $$\mathbf{y} = \mathbf{x}$$.

In less formal terms, this just means that $$S$$ is a bunch of individual points that don't touch each other.

#### Definition 2 (Additive subgroup).
A subgroup $$H \subseteq G$$ is additive if $$\mathbf{0} \in H$$, and $$-\mathbf{x}, \mathbf{x} + \mathbf{y} \in H$$ for every $$\mathbf{x}, \mathbf{y} \in H$$.

Essentially, we restrict the group operation to addition. Using the previous two definitions, we can now define a lattice.

#### Definition 3 (Lattice).
A lattice $$\mathcal{L}$$ is any discrete, additive subgroup of $$\mathbb{R}^n$$.

It turns out that every lattice $$\mathcal{L}$$ can be represented by a basis of linearly independent vectors. By taking all the linear combinations of these vectors such that the coefficients are integer, we obtain $$\mathcal{L}$$. While by no means trivial, this is a pretty intuitive result! Lattices are often represented in this way, in the form of a matrix. It's important to note that such a basis is *not* unique (why?).

## The sphere packing problem

My favorite introduction to the idea of a lattice is through the sphere packing problem. This is a classical problem, concerned with finding out how densely you can pack spheres of identical size. Spheres don't pack nicely. Whichever way you arrange them, there will be unused space. Nonetheless, you're probably familiar with the answer. If you've ever been to a grocery store, you'll have seen the way that oranges are stacked on the stand. That this is the optimal packing in three dimensions is called the [Kepler conjecture](https://en.wikipedia.org/wiki/Kepler_conjecture).

We naturally generalize this problem to other dimensions. In one dimension, this problem is easy. A one-dimensional sphere is simply a line segment. We can place each segment end to end, obtaining a packing that uses all of the space available. In two dimensions, we can take an infinite row of circles, and arrange it as closely as possible to another row. This results in a hexagonal pattern, in which approximately 0.91 of space is used. The method of stacking oranges in three dimensions is also called the face-centered cubic packing, and it uses a little over 0.74 of space.

While the results in the first two dimensions have long been known to be optimal, the Kepler conjecture was only proven in 1998! It took four years to peer review, and even then, there remained some doubt. It was only in 2014 that a complete formal proof was given. The sphere packing problem is *hard*.

If we draw points at the center of the spheres in all three packings, it turns out that we obtain lattices. That this is the case is remarkable, and not entirely intuitive! After all, there is no reason why the densest packing in a dimension should have group structure. Indeed, there are many dimensions in which the densest packing known is not a lattice at all (note the use of the word *known*, for they have not been proven optimal!). In this article we will focus on lattice packings, as they are generally more interesting.

## Lattice density

Since we're interested in maximizing packing density, let's take a look at how to calculate the density of a lattice. Let us, for the rest of this article, restrict ourselves to full rank lattices, i.e. those living in $$n$$ dimensions, and having $$n$$ linearly independent basis vectors. Suppose $$\mathcal{L}$$ has basis vectors $$\mathbf{B} = \{\mathbf{b}_1,\dotsc,\mathbf{b}_n\}$$. Consider the region defined by

$$ \theta_1\mathbf{b}_1 + \dotsb + \theta_n\mathbf{b}_n \quad (0 \leq \theta_i < 1) .$$

We call this region a *fundamental parallelotope* for $$\mathcal{L}$$. We can tile $$\mathbb{R}^n$$ with this region, which contains exactly one lattice point per copy. The determinant of a matrix can be viewed as the scaling factor of the transformation described. It follows, then, that $$
|\det \mathbf{B}|
$$ is equal to the volume of the fundamental parallelotope (think of the identity matrix, and the $$n$$-cube fundamental parallelotope it describes. When we transform this matrix by $$\mathbf{B}$$, this cube of volume 1 is scaled by $$
|\det \mathbf{B}|
$$). This volume is unique to $$\mathcal{L}$$, and not dependent on the basis. All we need now is the volume and radius of an $$n$$-sphere, and we can give a formula for the density of a lattice packing. First, however, we define a metric for how close together the points of a lattice are.

#### Definition 4 (Minimal distance.)
A shortest nonzero lattice vector defines the *minimal distance* of a lattice $$\mathcal{L}$$. Alternatively,

$$ \lambda_1(\mathcal{L}) = \min_{\mathbf{v} \in \mathcal{L} \setminus \{\mathbf{0}\}} \| \mathbf{v} \| .$$

#### Definition 5 (Density).
If we denote $$V_n$$ the volume of an $$n$$-sphere of radius 1, the density of the sphere packing given by the lattice $$\mathcal{L}$$ is

$$ \Delta = \left(\frac{\lambda_1}{2}\right)^n \frac{V_n}{|\det \mathcal{L}|} .$$

Often, we will leave out $$V_n$$. This is $$\delta$$, or the *center density* of the lattice. It can be interpreted as the average number of lattice points per unit volume. By scaling the lattice such that $$\lambda_1 = 2$$, we are left with only the inverse of the determinant. This gives us an easy way to compare the density of lattices.

The densest packing in one dimension is also the most trivial lattice there is -- the set of integers, $$\mathbb{Z}$$! It is easy to verify that this set is discrete, and also a group under addition. We can easily extend this lattice to $$n$$ dimensions, forming the integer lattice $$\mathbb{Z}^n$$. If we consider $$n$$-spheres of radius 1 (i.e. a lattice consisting of all points with even integer coordinates), we see that the center density (equal to $$2^{-n}$$) decreases rapidly with the dimension.

## Lattices packings in higher dimensions

Finding dense lattices in higher dimensions is no easy task. There is no standard formula or technique that leads to dense lattices in a given dimension. John Leech found his lattice in 1967 by studying binary error-correcting codes, and lifting one such code to $$\mathbb{Z}^{24}$$. By restricting the sum of the coordinates to zero modulo 4, and then sliding another copy of the resulting lattice into itself, he discovered what is now known as the Leech lattice.

Despite this difficulty, using a variety of techniques lattices in higher dimensions have been discovered and catalogued. Doing so revealed two very exceptional lattices. In eight dimensions, we have $$E_8$$, also known as the Gosset lattice. This lattice was discovered as early as the late 1800s, and has a very simple construction. We first define the lattice $$D_n$$ as follows:

$$ D_n = \{(x_1,\dotsc,x_n) \in \mathbb{Z}^n : x_1 + \dotsb + x_n \equiv 0 \mod 2 \} .$$

This lattice sequence, similar to that of the integer lattice, suffers from rapidly decreasing center density as $$n$$ grows. Big holes appear and increase in size. However, in eight dimensions, these holes are arranged in such a way that it is possible to slide another copy of $$D_8$$ into itself. This creates a new lattice with double the density but the same minimal distance! Formulaically,

$$ E_8 = D_8 \cup \left( \frac{1}{2}, \dotsc, \frac{1}{2} \right) + D_8 .$$

In twenty-four dimensions, we have the previously mentioned Leech lattice $$\Lambda_{24}$$. Constructions for the Leech lattice are more complex, and typically make use of other mathematical objects, such as the [Golay code](https://en.wikipedia.org/wiki/Binary_Golay_code). However, there is a nice inductive construction of the Leech lattice that highlights one of its special properties.

#### Definition 6 (Laminated lattice).
Let $$\Lambda_0$$ be the lattice in zero dimensions, consisting of a single point. Take the set of all $$n$$-dimensional lattices (for $$n \geq 1$$) with minimal distance 2 having at least one sublattice $$\Lambda_{n-1}$$ that is laminated. Any lattice having minimal determinant in this set is a laminated lattice.

We use $$\Lambda$$ to denote a laminated lattice. The above definition might be difficult to grasp, but it is actually a very intuitive way to construct a lattice. We start with the only laminated lattice in one dimension, $$\Lambda_1 = 2\mathbb{Z}$$. Drawing a row of unit circles centered around these points, we can obtain the hexagonal lattice $$\Lambda_2$$ (better known as $$A_2$$) by packing rows as close to each other as possible. Compactly layering the hexagonal lattice in three dimensions gives us the face-centered cubic lattice $$\Lambda_3 = D_3$$, which is equivalent to the familiar structure of stacked oranges on a fruit stand. In essence, we *laminate* infinite copies of the previous lattice into the new dimension.

For the first eight and also the twenty-fourth dimensions, the laminated lattices are unique, and also give optimal *lattice* packings. The unique $$\Lambda_{24}$$ is known as the Leech lattice! Its center density is exactly 1, which is higher than that of all known packings in lower dimensions. In general, laminated lattices are not necessarily unique, nor do they necessarily give the densest lattice packing in their dimension.

So what makes $$E_8$$ and $$\Lambda_{24}$$ so special? This question has many possible answers, as both lattices are intimately related to a large number of other branches of mathematics. Certainly, the why is more elusive, and not well understood. In the context of sphere packing, they are simply unexpectedly good packings.

Mathematicians have attempted to find tight upper bounds on the center density as long as the sphere packing problem has been studied in detail. Many of these bounds have been found and improved throughout the years. While the best known packings in nearly all dimensions fall well short of these bounds, $$E_8$$ and $$\Lambda_{24}$$ were shown to come extremely close. So close, that most were convinced of their packing optimality in general, in addition to being optimal among lattice packings. Remarkably, both lattices have the property that all densest lattice packings in lower dimensions can be found in their cross-sections. For the Leech lattice, this is especially interesting, as the densest lattices in some dimensions (such as 11, 12, and 13) are not laminated lattices! Indeed, some of these dense lattices were only discovered due to the discovery of $$E_8$$ and $$\Lambda_{24}$$.

Earlier this year, [two](https://arxiv.org/abs/1603.04246) [papers](https://arxiv.org/abs/1603.06518) were posted that finally prove $$E_8$$ and $$\Lambda_{24}$$ to be optimal sphere packings in their respective dimensions. Although there was little doubt left, it is very nice to finally have proof, and some sense of closure. The sphere packing problem has now been solved in one, two, three, eight, and twenty-four dimensions.

It remains an exceptionally difficult problem!

## Resources

[1] Gabriele Nebe and NJA Sloane. *Table of densest packings presently known*. [http://www.math.rwth-aachen.de/~Gabriele.Nebe/LATTICES/density.html](http://www.math.rwth-aachen.de/~Gabriele.Nebe/LATTICES/density.html).

[2] John H Conway and NJA Sloane. *Sphere packings, lattices, and groups*. SpringerVerlag, New York, 1993.
