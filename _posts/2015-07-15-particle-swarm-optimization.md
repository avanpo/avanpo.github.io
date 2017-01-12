---
title: "Particle swarm optimization"
categories: algorithms local-search
---

Particle swarm optimization (PSO) is a local search metaheuristic. It was initially developed by Kennedy, Eberhart and Shi in the late 1990s, and was meant to simulate social behaviour such as birds flocking to a promising position in a multi-dimensional space. A simplified version of the algorithm was found to perform optimization, which led to its adoptation as a local search algorithm.

The algorithm is easy to implement, and makes very few assumptions about the problem. PSO can be applied to a wide range of optimization problems, as it does not use the objective function's gradient and therefore does not require the objective function to be differentiable.

## Representation

The algorithm is represented by a population (swarm) of individuals (particles) that travel through the solution space, from iteration to iteration.

Each particle is a candidate solution, characterized by a position vector and a velocity vector in N-dimensional space.

$$ X_i(t) = \{ x_{i1}(t), \ldots, x_{iN}(t) \} $$

$$ V_i(t) = \{ v_{i1}(t), \ldots, v_{iN}(t) \} $$

The position bounds depend on the problem characteristics. Typically the velocity is bounded by the maximum of the position, in both directions. For example, if the position range is $$ [0,M] $$, then the velocity range will be $$ [-M,M] $$. In this way a particle traveling at maximum speed will traverse the entire solution space in a given dimension during one iteration.

## The algorithm

The algorithm is initialized with a population of particles, each given a uniformly distributed random starting position and velocity. 

During each iteration, each particles' position and velocity are updated according to the following formulas:

$$ V_i(t) = w(t) V_i(t-1) + c_1 r_1 (X_i^L - X_i(t-1)) + c_2 r_2 (X^G - X_i(t-1)) $$

$$ X_i(t) = V_i(t) + X_i(t-1) $$

The variables are defined as follows.

* $$ w(t) $$ is called the inertia weight, as it controls how easily a particle's current trajectory is changed
* $$ X_i^L $$ is the particle's best known position
* $$ X^G $$ is the swarm's (or a sub-swarm's) best known position
* $$ r_1, r_2 $$ are uniformly distributed random numbers between 0 and 1
* $$ c_1, c_2 $$ are the learning factors

The inertia weight and learning factors are set by the practitioner, and their selection is non-trivial. Trial and error or another overlaying optimizer can be used to find the best values.

These formulae potentially allow for a particle to move outside the position boundaries. To enforce particles to search inside the solution space during the algorithm, a variety of boundary conditions can be used. A very basic boundary condition adjusts out of bound values to the bound, but this may introduce bias. Reflecting and damping boundary conditions can be more effective and/or efficient. Often, this will depend on whether the global optimum lies close to the boundary or not.

The algorithm terminates after some condition is met. Some possible termination conditions include a maximum amount of iterations since the global best solution was last updated, or an overall maximum amount of iterations.

## How well does it work?

The basic PSO is sensitive to the coordinate system, so it is biased to solutions on an axis of the solution space. It also needs modification before it can be guaranteed to find a local optimum.

In a practical scenario, it probably isn't the first algorithm I would try. A better approach is to start simple. First I would try a hill climber, then a simple metaheuristic like simulated annealing, and only after that consider a population-based algorithm like PSO.

It is unclear whether PSO will yield better results than more popular population-based metaheuristics like genetic algorithms. Regardless, the technique has garnered enough interest to effect a significant amount of research.
