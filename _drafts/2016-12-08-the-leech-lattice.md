---
title: "The Leech lattice"
categories: lattices
---

<p class="preface">This article assumes basic knowledge of linear algebra and group theory.</p>

What is the Leech lattice? Good question. First, we'll need to define some concepts.

## Definitions

When we think of the word *lattice*, we most likely envision some kind of spacial structure. The faces of a crystal, or the metal beams of a bridge that form a repeating pattern of shapes. Using mathematics, we can distill this structure down to a repeating arrangement of points in some $$n$$-dimensional space.

There are many formal ways to define a lattice, some many pages long. Let's restrict ourselves to Euclidean vector spaces, and use group theory to simplify our definition. We start with the definition of a discrete subset.

#### Definition 1 (Discrete subset).
A subset $$S$$ of $$\mathbb{R}^n$$ is discrete if for each $$x \in S$$ there exists a positive real number $$\epsilon$$ such that the only vector $$y \in S$$ with $$\|x - y\| < \epsilon$$ is given by $$y = x$$.

In less formal terms, this just means that $$S$$ is a bunch of individual points that don't touch each other.

#### Definition 2 (Additive subgroup).
A subgroup $$H \subset G$$ is additive if $$\mathbf{0} \in H$$, and $$-\mathbf{x}, \mathbf{x} + \mathbf{y} \in H$$ for every $$\mathbf{x}, \mathbf{y} \in H$$.

Essentially, we restrict the group operation to addition. Using the previous two definitions, we can now define a lattice.

#### Definition 3 (Lattice).
A lattice $$\mathcal{L}$$ is any discrete, additive subgroup of $$\mathbb{R}^n$$.

It turns out that every lattice $$\mathcal{L}$$ can be represented by a basis of linearly independent vectors. By taking all the linear combinations of these vectors such that the coefficients are integer, we obtain $$\mathcal{L}$$. While by no means trivial, this is a pretty intuitive result! Lattices are often represented in this way, in the form of a matrix. It's important to note that such a basis is *not* unique (why?).

## The sphere packing problem

My favorite introduction to the idea of a lattice is through the sphere packing problem. This is a classical problem, concerned with finding out how densely you can pack spheres of identical size. Spheres don't pack nicely. Whichever way you arrange them, there will be unused space. Nonetheless, you're probably familiar with the answer. If you've ever been to a grocery store, you'll have seen the way that oranges are stacked on the stand. That this is the optimal packing is called the [Kepler conjecture](https://en.wikipedia.org/wiki/Kepler_conjecture).

We naturally generalize this problem to other dimensions. In one dimension, this problem is easy. A one-dimensional sphere is simply a line. We can place each line end to end, and obtain full coverage of space, as compared to a little over 0.74 for the three dimensional case. In two dimensions, we obtain a hexagonal pattern.

While the results in the first two dimensions have long been known to be optimal, the Kepler conjecture was only proven in 1998! It took four years to peer review, and even then, there remained some doubt. It was only in 2014 that a complete formal proof was given. The sphere packing problem is *hard*.

If we draw points at the center of the spheres in all three packings, we obtain lattices. That this is the case is remarkable, and not entirely intuitive! After all, there is no reason why an irregular packing shouldn't be more dense.

## Lattice packing density

Since we're interested in maximizing density, let's take a look at how to calculate the density of a lattice. Let us, for the rest of this article, restrict ourselves to full rank lattices, i.e. those living in $$n$$ dimensions, and having $$n$$ linearly independent basis vectors. Suppose $$\mathcal{L}$$ has basis vectors $$\mathbf{B} = \{\mathbf{b}_1,\dotsc,\mathbf{b}_n\}$$. Consider the region defined by

$$ \theta_1\mathbf{b}_1 + \dotsb + \theta_n\mathbf{b}_n \quad (0 \leq \theta_i < 1) .$$

We call this region a *fundamental parallelotope* for $$\mathcal{L}$$. We can tile $$\mathbb{R}^n$$ with this region, which contains exactly one lattice point per copy. The determinant of a matrix can be viewed as the scaling factor of the transformation described. It follows, then, that $$\det \mathbf{B}$$ is equal to the volume of the fundamental parallelotope (think of the identity matrix, and the $$n$$-cube fundamental parallelotope it describes. When we transform this matrix by $$\mathbf{B}$$, this cube of volume 1 is scaled by $$\det \mathbf{B}$$). This volume is unique to $$\mathcal{L}$$, and not dependent on the basis. All we need now is the volume and radius of an $$n$$-sphere, and we can give a formula for the density of a lattice packing.

#### Definition 4 (Density).
If we denote $$V_n$$ the volume of an $$n$$-sphere of radius 1, and $$\lambda_1$$ the minimal distance between the lattice points of $$\mathcal{L}$$, the density of the packing given by $$\mathcal{L}$$ is

$$ \Delta = \left(\frac{\lambda_1}{2}\right)^n \frac{V_n}{\det \mathcal{L}} .$$

Often, we will leave out $$V_n$$. This is $$\delta$$, or the *center density* of the lattice. It can be interpreted as the average number of lattice points per unit volume. By scaling the lattice such that $$\lambda_1 = 2$$, we are left with only the inverse of the determinant. This gives us an easy way to compare the density of lattices.

The densest packing in one dimension is also the most trivial lattice there is. The set of integers, $$\mathbb{Z}$$! It is easy to verify that this set is discrete, and also a group under addition. We can extend this to $$n$$ dimensions, to form the integer lattice $$\mathbb{Z}^n$$. If we consider $$n$$-spheres of radius 1, we see that the center density (equal to $$2^{-n}$$) decreases rapidly with the dimension.

## Lattices in higher dimensions

Finding dense lattices in higher dimensions is no easy task. There is no standard formula or technique that leads to dense lattices in a given dimension. John Leech found his lattice in 1967 by studying binary error-correcting codes, and lifting one such code to $$\mathbb{Z}^n$$. By restricting the sum of the coordinates to zero modulo 4, and then sliding another copy of the resulting lattice into itself, he discovered what is now known as the Leech lattice.


