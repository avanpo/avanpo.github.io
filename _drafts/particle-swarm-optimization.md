---
title: "Particle Swarm Optimization"
categories: algorithms local-search
---

Particle swarm optimization (PSO) is a local search metaheuristic. It was initially developed by Kennedy, Eberhart and Shi in the late 1990s, and was meant to simulate social behaviour such as birds flocking to a promising position in a multi-dimensional space. A simplified version of the algorithm was found to perform optimization, which led to its adoptation as a local search algorithm.

The algorithm is easy to implement, and makes very few assumptions about the problem. PSO can be applied to a wide range of optimization problems, as it does not use the objective function's gradient and therefore does not require the objective function to be differentiable.

## Representation

The algorithm is represented by a population (swarm) of individuals (particles) that travel through the solution space, from iteration to iteration.

Each particle is a candidate solution, characterized by a position vector and a velocity vector in N-dimensional space.

$$ X\_i(t) = \{ x_{i1}(t), \ldots, x_{iN}(t) \} $$

$$ V\_i(t) = \{ v_{i1}(t), \ldots, v_{iN}(t) \} $$

The position bounds depend on the problem characteristics. Typically the velocity is bounded by the maximum of the position, in both directions. For example, if the position range is \\( [0,M] \\), then the velocity range will be \\( [-M,M] \\). In this way a particle traveling at maximum speed will traverse the entire solution space in a given dimension during one iteration.

## The algorithm

The algorithm is initialized with a population of particles, each given a uniformly distributed random starting position and velocity. 

During each iteration, each particles' position and velocities are updated according to the following formulas.

$$ V\_i(t) = w(t) V\_i(t-1) + c\_1 r\_1 (X\_i^L - X\_i(t-1)) + c\_2 r\_2 (X^G - X\_i(t-1)) $$

$$ X_i(t) = V_i(t) + X_i(t-1) $$

The coefficients \\( r\_1 \\) and \\( r\_2 \\) are uniformly distributed random numbers between 0 and 1. \\( w(t) \\) is called the inertia weight, as it controls how easily a particle's current trajectory is changed. \\( c\_1 \\) and \\( c\_2 \\) correspond to learning factors. These last three parameters are set by the practitioner, and their selection is non-trivial. Trial and error or another overlaying optimizer can be used to find the best values.

// more stuff about X^L and X^G

These formulae potentially allow for a particle to move outside the position boundaries. To enforce particles to search inside the solution space during the algorithm, a variety of boundary conditions can be used. A very basic boundary condition adjusts out of bound values to the bound, but this may introduce bias. Reflecting and damping boundary conditions can be more effective and/or efficient. Often, this will depend on whether the global optimum lies close to the boundary or not.

The algorithm terminates after some condition is met. Some possible termination conditions include a maximum amount of iterations since the global best solution was last updated, or an overall maximum amount of iterations.

## How good is it?

// stuff about what can be improved

// probably wouldn't use it before trying hill climber, simulated annealing, GA in that order
