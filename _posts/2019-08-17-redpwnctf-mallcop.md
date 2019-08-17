---
title: "RedpwnCTF: mallcop"
categories: ctf algo
---

Mallcop was the least-solved challenge in the Misc category. We were given a
zipped file of 20 problem instances. The server gives us the following problem
description:

```txt
Sick of Chick-fil-A, Daniel's son deserts him to make a break for the parking
lot. (Un?)fortunately, Max the Mall Cop is here to help!  The n stores in the
mall are connected by walkways such that none of them form a cycle; in other
words, there is exactly one path from one store to another. Daniel's son was
last seen at store k.  Stores connected to only one store lead to a parking lot.
Max the Mall Cop decides to station his miniature clones at some of these
parking lot stores.  Every minute, Daniel's son and the miniature Maxes can move
one meter towards an adjacent store or stay put. Daniel's son is caught when he
coincides with a miniature Max at a store or a walkway.  What is the minimum
number of miniature Maxes that Max the Mall Cop should station to ensure that
Daniel's son cannot reach a parking lot?

Input:
    First line: n, k
    Next n-1 lines: a, b, x indicating a walkway between stores a and b with
        length x meters

Output:
    One line with the minimum miniature Maxes needed.

Download the testcases from in.zip and enter your output here.
```

The server has us give the answer to each of the 20 instances one at a time, and
then disconnects us without giving any information if we got it wrong.

Let's take a closer look at the problem. Note that there is exactly one path
from one store to another. This means that we can think of the mall as an
undirected, connected acyclic graph. This is also known as a [spanning
tree](https://en.wikipedia.org/wiki/Spanning_tree).

Let's read the file and build a graph. Let's also get a list of parking lot
stores.

```python
with open('%s.in' % instance, 'r') as f:
    lines = f.readlines()

n, k = map(int, lines[0].strip().split(' '))
edges = collections.defaultdict(list)

for line in lines[1:]:
    a, b, x = map(int, line.strip().split(' '))
    edges[a].append((b, x))
    edges[b].append((a, x))

p_stores = [s for s in edges.keys() if len(edges[s]) == 1]
```

Spanning trees have a nice property which lends itself nicely to this problem,
namely the notion of a fundamental cutset. A single miniature Max anywhere in
the mall will partition it into two, keeping Daniel's son on one side. Moving
that mini Max closer to Daniel's son is guaranteed to decrease or leave
unaltered the number of stores and parking lots in Daniel's son's partition. It
also doesn't matter whether the mini Max is at a store, or somewhere along the
(unique) edge from that store toward Daniel's son; the partition will remain the
same (until the mini Max reaches the next store).

So the problem becomes: which stores can the mini Maxes be guaranteed to reach
before Daniel's son, and which of those are at the border?

Let's calculate how long it'll take a mini Max (ok wow, I only just got the
[reference](https://en.wikipedia.org/wiki/Minimax)) to reach a given store.

```python
def annotate(store, curr_dist, edges, distances):
    if store in distances and distances[store] <= curr_dist:
        return
    distances[store] = curr_dist
    for next_store, step_dist in edges[store]:
        annotate(next_store, curr_dist + step_dist, edges, distances)

minimax_dists = {}
for p in p_stores:
    annotate(p, 0, edges, minimax_dists)
```

This uses a depth first search to visit the stores from each parking,
calculating the shortest distance encountered.

Now we need to figure out which stores are on the border, e.g. those that will
box Daniel's son in to a section of the mall.

```python
def count_border_stores(store, curr_dist, edges, son_dists, minimax_dists):
    if store in son_dists:
        return 0
    son_dists[store] = curr_dist
    if minimax_dists[store] <= curr_dist:
        # This is a border store. A mini Max can get to this store earlier or
	# simultaneously and block off the mall beyond this.
        return 1
    count = 0
    for next_store, step_dist in edges[store]:
        count += count_border_stores(next_store, curr_dist + step_dist, edges,
	        son_dists, minimax_dists)
    return count

print(count_border_stores(k, 0, edges, {}, minimax_dists))
```

This is the answer to the problem instance. Placing a mini Max at each of these
stores would block Daniel's son in completely. They are guaranteed to get there
first. Any improvement would require a mini Max at a store that has two or more
border stores behind it. This would necessarily mean that Daniel's son could
reach this store first, and that there would be at least two separate paths
toward a parking lot. A single mini Max could not possibly cover both.

That's it! This challenge would have been a lot more difficult if the graphs
included cycles.
