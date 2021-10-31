<p align="center">
      <a href="https://scott-hamilton.mit-license.org/"><img alt="MIT License" src="https://img.shields.io/badge/License-MIT-525252.svg?labelColor=292929&logo=creative%20commons&style=for-the-badge" /></a>
</p>
<h1 align="center">trees-with-n-nodes-stanza - Stanza's implementation of the trees with n nodes problem</h1>

## Building
This project is configured with stanza's build system
```shell_session
$ stanza build
```

## Usage
My stanza version doesn't yet have the arg-parsing package.
So in the mean time, modify the src/main.stanza file to your needs and recompile.
You can, change the number of nodes per tree and print the tests, or anything else.

## How does it work ?
### trees-with-n-nodes
The most important function is this one:
```clojure
public defn trees-with-n-nodes (N: Int) :
    generate<Tree<String>> :
        for root-children in in-reverse(1 to N) do :
            val tree = make-tree(root-children)
            val left = N - root-children - 1
            if left == 0 :
                yield(tree)
            else if left == 1 :
                set-child(tree, root-children - 1, make-tree(1))
                yield(tree)
            else if left > 1 :
                val repartitions = repart(left, root-children)
                for repartition in repartitions do :
                    for t in repartition apply-repartition:
                        yield(t)
```
`trees-with-n-nodes` generates all possible trees with N nodes.
For example, `trees-with-n-nodes(4)` would generate those **final trees**
(don't mind the letters and the numbers, it's irrelevant):
```
1.       2.           3.            4.
R        R            R             R
├── 1    ├── 1        └── R         └── R
├── 2    └── R            ├── 1         └── 1
└── 3        └── 1        └── 2             └── 1
```
The first loop
`for root-children in in-reverse(1 to N) do :`
iterates through all possible numbers of children that can have the root node.
On the last example, the values taken by root-children would have been `3, 2 and 1`.
The maximum number of children the root node can have is `N-1`, and the minimum is 1.

```clojure
val tree = make-tree(root-children)
```
For each iteration, we create a *base-tree* with the root-children number of children on the root node.
Following the previous example, the different *base-tree*'s would have looked like this:
```
1.       2.           3.
R        R            R
├── 1    ├── 1        └── R
├── 2    └── 2
└── 3
```
As you can see, there are only three *base-tree* to work on, but there are 4 **final trees**.
Which means that 2 **final trees** originated from the last *base-tree*, in our example, those are
the **final trees** 3 and 4.
```clojure
val left = N - root-children - 1
```
This next line associates to each *base-tree* a *left* counter which is the left-number of nodes that
we need to somehow add to the *base-tree* in order to get a valid **final-tree**.
The first *base-tree* doesn't need any more nodes, it's *left* value is 0, the second needs one more node, it's *left* value is one, and the last one needs 2 nodes, it's *left* value is 2.
`0, 1, 2`.

The next lines are quite easy to understand, they look at the *left* value of the current *base-tree* and
act consequently.
```clojure
if left == 0 :
	yield(tree)
```
A *left* value of 0 means that the *base-tree* is already a **final tree**. So we can yield it.
This is the case for the first **final tree** of our example.

```clojure
else if left == 1 :
	set-child(tree, root-children - 1, make-tree(1))
	yield(tree)
```
A *left* value of 1 means that the *base-tree* only needs 1 node to become a **final tree**.
By convention, we add the node to the last child-node of the *base-tree*.
This is the case for the second **final tree** of our example. 

And here starts the interesting part. What should we do with a *left* value greather than 1 ?
Or in other words, what are all the possible ways to add 2, 3, 4, 5... nodes to a tree. Here is the code in charge of that:
```clojure
else if left > 1 :
	val repartitions = repart(left, root-children)
	for repartition in repartitions do :
		for t in repartition apply-repartition:
			yield(t)
```
We see that the solution involves a "repart" function, let's check what it does.
```clojure
for r in repart(4, 4) :
	println("%_" % [r])
```
Outputs:
```
(0 0 0 4)
(0 0 1 3)
(0 0 2 2)
(0 1 1 2)
(1 1 1 1)
```
This repart function takes a first argument, the *left* value,
and a second argument the number of children on the root node.
And it returns all the possible repartitions of nodes per root-child.
The first repartition means: give 4 nodes to the last root-child and nothing to the others.
The last repartition means: give 1 node to all the root-children.

In our example, the third *base-tree* had a *left* value of 2, and a root-children value of 1,
so the repart function was called with:
```clojure
repart(2, 1)
```
Which simply outputs:
```
(2)
```
Meaning that we (obviously) have to add 2 nodes to the only child of the root node.
Then we loop through all repartitions and we call this mysterious "apply-repartition" function.
Finally we loop through all the trees generated by this function and yield them too. (I guess they would
use yield from in python)
```clojure
for repartition in repartitions do :
	for t in repartition apply-repartition:
		yield(t)
```

### apply-repartition
The second most important function is this one:
```clojure
public defn apply-repartition (repartitions: List<Int>) :
	[...] ; those lines aren't very important
    val groups: List<List<Int>> = map(fn (g) : map({_[1]}, g), to-list(tmp-groups))
```
The only argument it takes is the repartitions, let's say `(0 0 2 2)`.
It doesn't even need the *base-tree* to work, it can deduce it from the repartitions.
The first thing this function does is grouping the repartitions together by value.
The first few lines are just there to keep the order when grouping
to avoid that for example this repartition:
`(1 2 2 3)` (which isn't realistic but let's pretend) is grouped this way:
`((2 2) (1) (3))`. The order doesn't really matter at the end but we would like to keep it
the closest to the original repartition because the order corresponds to which child will become what.
So we would prefer the previous repartition to be grouped this way:
`((1) (2 2) (3)`.
Anyway, the reason behind grouping the repartition by value is that if you don't do that, you'll eventually end up generating **final trees** with the same topological structure which is wrong, we'll see an example of that [later](#side-note-on-why-grouping).

Now that we have groups, we loop though each group: 
```clojure
    for g in groups do :
        val r = g[0]
		[...]
```
And we retrieve common "r" value of the group which is the number of nodes to add to each child.
This is better understood with an example, let's keep the same repartition as above `(1 2 2 3)`.
From this, we can deduce that the original *base-tree* looked like this (I named each root-children for clarity):
```
R
├── A
├── B
├── C
└── D
```
The grouped partition would look like that: `((1) (2 2) (3))`.
The first group `(1)` has an r value of 1, and corresponds to the child `A`.
The second group `(2 2)` has an r value of 2 and corresponds to the children `B` and `C`.
The last group `(3)` has an r value of 3 and corresponds to the child `D`.
So you see that each grouped repartition value can be directly mapped to a child by reading it in depth-first order. Here is a table that sums it up:

| Group | left value | Corresponding children |
|-------|------------|------------------------|
| (1)   | 1          | A                      |
| (2 2) | 2          | B and C                |
| (3)   | 3          | D                      |

Next we compute for each group the corresponding `group-c-ps` which is a list of all the possible trees with r+1 nodes .
```clojure
val group-c-ps = [...]
```
Let's consider the first group `(1)`. It's r = 1 so its associated `group-c-ps` corresponds to
all the possible trees with r+1=2 nodes. Which is exactly what gives us the `trees-with-n-nodes` function.
```clojure
val group-c-ps = if r == 0 :
	to-array<Tree<String>>([ make-tree(0) ])
else :
	to-array<Tree<String>>(trees-with-n-nodes(r + 1))
```
Note that we make 2 different cases because the `trees-with-n-nodes` function doesn't work well on trees with 1 node.

Now we have this `group-c-ps` list which is like a catalog of all the possible trees that we could use.
But now we need to associate them together.

Let's stop talking with trees for a minute and say that the `group-c-ps` catalog is a car catalog:
`(BMW PEUGEOT VOLKSWAGEN)`. And let's pretend that the group looked like `(2 2)`. The r value of the group is 2. The catalog contains 3 different car models.
Now we have to generate all the possible combinations of two cars in the catalog. A combination (BMW PEUGEOT) is considered the same as a combination (PEUGEOT BMW).
For our example all the possible combinations would look like:
```
(BMW BMW)
(BMW PEUGEOT)
(BMW VOLKSWAGEN)
(VOLKSWAGEN VOLKSWAGEN)
(VOLKSWAGEN PEUGEOT)
(PEUGEOT PEUGEOT)
```
This is exactly what the `combinations-with-replacement` function does.
Check out it's definition in the [python's itertools doc](https://docs.python.org/3/library/itertools.html#itertools.combinations_with_replacement).

Now let's get back to our trees. We compute all the possible combinations of n-trees from our catalog
where n is the number of items in the group.
In our previous example, the group was `(2 2)` so r=2+1=3, the catalog corresponds to all trees
with 3 nodes:
```
Tree A:
R
├── 1
└── 2
Tree B:
R
└── R
    └── 1
```
The length of the group is 2, meaning there are two values in it. So we compute all the combinations of
A-Tree's and B-Tree's: 
```clojure
val ps: List<List<Tree<String>>> = to-list(combinations-with-replacement(group-c-ps, length(g)))
```
`ps` (which stands for possibilities) is equal to
```
(
	(A A)
	(A B)
	(B B)
)
```
Now that we have computed all the possible combinations for each group. We collect them together in a final list called `groups-ps`:
```clojure
add(groups-ps, ps)
```
With a grouped repartition like so: `((1) (2 2) (3))`, the final `groups-ps` would like like that:
```
(
	((A1))

	((A2 A2)
	 (A2 B2)
	 (B2 B2))

	((A3)
	 (B3)
	 (C3))
)
```
Where
```
A1 is the only possible tree with 1+1=2 nodes.
A2 and B2 are the only possible trees with 2+1=3 nodes.
A3, B3 and C3 are the only possible trees with 3+1=4 nodes.
```
And we're not done yet... Now we make all possible combinations for all catalog-combination-possibilities of each group, yes it's hard to explain.
This is done with the `all-combinations` function and the resulting combinations for our example looks like that (BTW, you must have noticed by now that I love stanza's Operating Functions):
```clojure
for combination in to-list(groups-ps) all-combinations :
```
```
((A1) (A2 A2) (A3))
((A1) (A2 A2) (B3))
((A1) (A2 A2) (C3))
((A1) (A2 B2) (A3))
((A1) (A2 B2) (B3))
((A1) (A2 B2) (C3))
((A1) (B2 B2) (A3))
((A1) (B2 B2) (B3))
((A1) (B2 B2) (C3))
```
And here we start to see the end of the tunnel, this looks very much like a list of children, except that we need to flatten everything up:
```clojure
val childs = to-list(lazy-flatten(combination))
```
We get this flattened list of children
```
(A1 A2 A2 A3)
(A1 A2 A2 B3)
(A1 A2 A2 C3)
(A1 A2 B2 A3)
(A1 A2 B2 B3)
(A1 A2 B2 C3)
(A1 B2 B2 A3)
(A1 B2 B2 B3)
(A1 B2 B2 C3)
```
So we have a list of children, let's take the first one of this example `(A1 A2 A2 A3)`.
This means that the corresponding **final tree** will have A1 as subtree for his first child,
A2 for his second child and A3 for his last child.
We know that A1 is the only possible tree with 2 nodes.
We know that A2 is the first possible tree with 3 nodes.
And we know that A3 is the first possible tree with 4 nodes.
We have:
```
A1:     A2:     A3:
R       R       R
└── 1   ├── 1   ├── 1
        └── 2   ├── 2
                └── 3
```
So applying this list of children `(A1 A2 A2 A3)` would result in this **final tree**:
```
R
├── R
│   └── 1
├── R
│   ├── 1
│   └── 2
├── R
│   ├── 1
│   └── 2
└── R
    ├── 1
    ├── 2
    └── 3
```
Then we give each child list to the `apply-child-possibility` function and we yield the returned tree.
```clojure
yield(apply-child-possibility(childs))
```

And this is how this algorithm works.

## Side note on why grouping
When first implementing this algorithm in python I wasn't grouping the
repartitions by value. But when I started implementing it in Rust cf [Implementiation in Rust](https://github.com/SCOTT-HAMILTON/trees_with_n_nodes), I realized that this could lead to topologically-equal generated trees.
For example, let's take the previous repartition `(1 2 2 3)`.
The previous algorithm would have generated those two **final trees** among others:
```
1.
R
├── R
│   └── 1
├── R
│   └── R
│       └── 1
├── R
│   ├── 1
│   └── 2
└── R
    ├── 1
    ├── 2
    └── 3
2.
R
├── R
│   └── 1
├── R
│   ├── 1
│   └── 2
├── R
│   └── R
│       └── 1
└── R
    ├── 1
    ├── 2
    └── 3
```
As you can see, these 2 trees are topologically-equal, because if you exchange the root-children 2 and 3 of the second tree, you get the first tree.

This is because these two **final trees** correspond to these unflattened combinations:

`1.((A1) (B2 A2) (A3)) and 2.((A1) (A2 B2) (A3))`.

Which means that that among all the generated combination possibilities of the group `(2 2)`, there were the possibilities
```
ps = 
(
 ...
 (B2 A2)
 ...
 (A2 B2)
 ...)
```
But remember that `ps` is as follow:
```clojure
val ps: List<List<Tree<String>>> = to-list(combinations-with-replacement(group-c-ps, length(g)))
```
And `combinations-with-replacement` would never generate those combinations.
So we can safely say for sure that the grouped algorithm fixed this issue.

## trees-with-n-nodes-with-m-leaves
This is the real final problem.
It's the same as the previous problem but we add a condition on the
number of leaf-nodes that the **final-trees** should have, the **final-trees**
should have **M** leaf-nodes.

Here is an example of a tree with `N=8 and M=5` or in other words
8 nodes including 5 leaf-nodes: 
```
R
└── R
    ├── 1
    │   ├── 1
    │   ├── 2
    │   ├── 3
    │   └── 4
    └── 2
```
In total there are 21 trees with 8 nodes including 5 leaf-nodes.

The main function responsible for this solution is here:
```clojure
public defn trees-with-n-nodes-with-m-leaves (N: Int, M: Int) :
    generate<Tree<String>> :
        val internal-N = N - M
        for internal-tree in trees-with-n-nodes(internal-N, M + 1) do :
            for final-tree in internal-tree-to-final-trees(internal-tree, M) do:
                yield(final-tree)
```
It first generates what I call **internal-trees**.
Let's take the last example `N=8 and M=5`.
The problem is thought as follows.
We know that we need to have `M` leaf-nodes at the end.
So we place them at the bottom, laying.
And what are we left with? Well 8 total nodes. Meaning that we know for sure
that there will be `N-M` **non-leaf-nodes**. In our example there will be
8-5=3 **non-leaf-nodes** at the end. And this is basically our **internal-trees**.
We know that we have to take in consideration all (actually not all but we will see that in the optimization section) **internal-trees**.
In this example there are only 2 possible **internal-trees** because there exists only 2 possible
trees with 3 nodes:
```
1.      2.
R       R
├── 1   └── R
└── 2       └── 1
```
As you can tell, there was a reason why we first solved the `trees-with-n-nodes` problem before
solving the final problem, and this is because we use the first solution in our final solution.
Then comes the harder part of the problem, which is to find a way to include all `M` leaf-nodes laying
around at the bottom and attach them to each **internal-tree** to get a **final-tree**. And we even have to
find all the possible ways to attach them to each **internal-tree**.

Here is a few examples of attaching the 5 leaf-nodes to the first **internal-tree**:
```
R            R            R
├── 1        ├── 1        ├── 1
│   ├── A    │   ├── A    │   ├── A
│   ├── B    │   ├── B    │   └── B
│   └── C    │   └── C    ├── 2
└── 2        ├── 2        │   └── C
    ├── D    │   └── D    ├── D
    └── E    └── E        └── E
```
Note that we can even attach some nodes to the root node.
And this introduction brings us to the second most important function
of the solution:
```clojure
public defn internal-tree-to-final-trees (internal-tree: Tree<String>, M: Int) :
    generate<Tree<String>> :
		[...]
		; Part 1
        val non-leaf-nodes = non-leaf-nodes(internal-tree)
        val leaf-nodes = leaf-nodes(internal-tree)
		[..]
        val min-nodes = length(leaf-nodes)
        val left = M - min-nodes
		[...]
		; Part 2
        defn aup-and-groupby (nodes-list: List<Tree<String>>) -> List<List<List<Long>>> :
			[...]
        ; println("Leaf Nodes: %_" % [leaf-nodes])
        val leaf-node-groups = aup-and-groupby $ leaf-nodes
        val non-leaf-node-groups = aup-and-groupby $ non-leaf-nodes
		[...]
```
I divided this gigantic function into parts to better understand it and cut the end where it gets really messy.
This function takes an **internal-tree** and M as arguments, and it generates all possible **final-trees**, or in other words,
all the possible ways to attach the leaf-nodes to the **internal-tree**.

The base of the solution relies in the separation between leaf-nodes **of** the **internal-tree**
(not leaf-nodes **of** the **final-tree**) and non-leaf-nodes **of** the **internal-tree**.
To be sure that we are in phase, let's say that leaf-nodes are rendered as an L and non-leaf nodes
are rendered as an N and let's print the **internal-trees** from above with this convention:
```
1.      2.
N       N
├── L   └── N
└── L       └── L
```
Next we enumerate each node so that we can differentiate them.
We drop the second **internal-tree** to focus on the first one.
```
1.
N1
 ├── L1
 └── L2
```
If we wanted to store the non-leaf-nodes and the leaf-nodes into two lists, those lists would look like that:
```clojure
val non-leaf-nodes = List( N1 )
val leaf-nodes = List( L1, L2 )
```
The most important rule when attaching the **final** leaf-nodes to the **internal-trees**
(please note the **final**), is that a leaf-node **of** the **internal-tree** should have at least
1 **final** leaf-node attached to it.

This is because if, for example, you only attach the **final** leaf-nodes to non-leaf-nodes of the **internal-tree**, then you end up with a **final-tree** that has more than M leaf-nodes, because it has all the 
**final** leaf-nodes but also all the leaf-nodes of the **internal-tree** that didn't get anything attached to them.
Let's take the first **internal-tree** and let's attach all the **final** leaf-nodes to the only non-leaf-node
**of** this **internal-tree**. The resulting **final-tree** will look like that:
```
N1
 ├── L1
 ├── L2
 ├── A
 ├── B
 ├── C
 ├── D
 └── E
```
You immediatly see that because we didn't attach anything to the L-nodes, they get counted as final leaf-nodes
and this **final-tree** is invalid.
In reality, each time you forget to attach something to an L-node (let's call them like that, it's easier
than "leaf-nodes **of** the **internal-tree**"), you'll end up with a **final-tree** with one more extra **final** leaf-node.
In our example, we forgot to attach something to both L-nodes so we ended up with a tree that has 2 extra
**final** leaf-nodes.

That's the reason why we separate L-nodes and N-nodes, because N-nodes can be forgotten without it having any
consequences on the number of **final** leaf-nodes.

The Part 1 just stores all the non-leaf-nodes of the **internal-tree** into an array of nodes.
We do the same with the leaf-nodes.
```Clojure
val non-leaf-nodes = non-leaf-nodes(internal-tree)
val leaf-nodes = leaf-nodes(internal-tree)
```

```clojure
val min-nodes = length(leaf-nodes)
val left = M - min-nodes
```
Then the first line above gets the number of L-nodes which we now know is the number of
**final** leaf-nodes for which we already know the destination.
And it calculates the left number of laying leaf-nodes for which we will have to work on a
destination.

The second part of this function uses what I call the `ascendant-unique-path` of a node.
This is a way of indexing each node with a unique identifier in the tree.
The AUP in short is a list of indices that starts from the node and moves upward
to the root node. Each index corresponds to the index of the child-node the path
took to get to the root node.

For example, consider this tree:
```
R
├── A
├── B
├── C
├── D
└── E
```
The node C is the third child of it's parent: the root-node R.
It's AUP is `(2)`.
Here is a table to sum up all the AUPs:

| Node | AUP |
|------|-----|
| R    | ()  |
| A    | (0) |
| B    | (1) |
| C    | (2) |
| D    | (3) |
| E    | (4) |

You see that by convention, every root-node have an AUP equal to the empty list ().

But the AUP is really useful to index nodes that are deeply hidden in a tree. Let's consider this tree:
```
R
├── A
├── B
├── C
└── D
    └── E
        ├── F
        └── G
```
The F-node is the first child of the E-node.

The E-node is the first and only child of the D node.

And the D-node is the fourth child of the root-node R.

So AUP of the F-node will be `(0 0 3)`
And you've guested it, the G-node's AUP will be `(1 0 3)`.
From bottom to top.

And please note that the AUP indexing method doesn't take into consideration the value of each node, so
if we consider this identic tree but with different node-values:
```
1
├── 2
├── 3
├── 4
└── 5
    └── 6
        ├── 7
        └── 8
```
We realize that the 7-node will have the same AUP as the F-node of the previous tree, same for the
8-node and the G-node.

The inverse of the AUP is the DUP which stands for the Descendant Unique Path which is basically computed by
reading the AUP in reverse, here is the DUP of the G-node: `(3 0 1)`.

The reason why we have both DUPs and AUPs is simply that it's easier to compute directly the AUP of a node and convert it to a DUP.
And the other way around, it's easier to take a tree and a DUP and find the corresponding Node.

```clojure
	; Part2
	defn aup-and-groupby (nodes-list: List<Tree<String>>) -> List<List<List<Long>>> :
		[...]
	; println("Leaf Nodes: %_" % [leaf-nodes])
	val leaf-node-groups = aup-and-groupby $ leaf-nodes
	val non-leaf-node-groups = aup-and-groupby $ non-leaf-nodes
```
Here is the second part of the function.
We first group the leaf-nodes into groups of leaf-nodes that have the same parent, we
call it siblingness-grouping. And once we have our groups, we don't actually keep those
node objects but only there AUP index.
And we do the same for the non-leaf-nodes.

So we end up with two lists of groups of AUP's which get's already quite ugly looking when printed out.
Let's take the first **internal-tree** of our example and compute the corresponding two lists for it.
```
N1
 ├── L1
 └── L2
```
We get:
```
leaf-node-groups = ( ((1) (0)) )
```
There is a single group because both L-nodes share the same parent: the root-node N1.
This single group contains the two L-nodes
```
L1's AUP=(0)
L2's AUP=(1)
The group: (L2 L1)=((1) (0))
The final list of groups: ( ((1) (0)) )
```
And for the non-leaf-nodes:
```
non-leaf-node-groups = ( (()) )
```
There is a single N-node, so only one group.
```
N1's AUP=() because it's the root-node
The group: (N1)=(())
The final list of groups: ( (()) )
```

The third part of the function looks like that:
```clojure
	for r in repart-unordered(left, 2) do:
		[...]
```
Now that we have grouped the nodes of the **internal-tree**,
And that we know that there are left=3 leaf-nodes to attach, how do we
tell which node we attach them to ?

The next part implements an algorithm that associates to each node of the **internal-tree**
a number of **leaf-node** that will be attached to it, this number doesn't include the 
leaf-nodes that will be automatically attached to every L-node.
The details of how this works are quite challenging to put into text so let's just see the
main function that allowed to solve this.
This is the `repart-unordered` function: 
```clojure
public defn repart-unordered (left: Int, N: Int) :
    generate<List<Int>> :
        for r in repart(left, N) do :
            for p in r heap-augmented :
                yield(p)
```
Similarly to the `repart` function seen in the [previous section](#trees-with-n-nodes),
it takes a left value, an N and generates all possible repartitions of N numbers
which sums up to the left value. Except that this time, the order counts and
this repartition: `(0 0 1)` is different from this repartition `(1 0 0)`.
Here is an example that showcases both functions:
```
repart(4, 2):
(0 4)
(1 3)
(2 2)
repart-unordered(4, 2):
(0 4)
(4 0)
(1 3)
(3 1)
(2 2)
```

The next lines are just there to propagate the repartition
of the leaf-nodes to both the L-nodes and the N-nodes.
And then we propagate the repartition between each group.
Finally each group's repartition is propagated with the
`repart` function, not the `repart-unordered`, this is important
and this is the only reason why we bother with groups. See the
[Side note on why grouping](#side-note-on-why-grouping) for more details
on this.

In the end we get list of Combination-Action-List. I apologise for this
barbaric name but I didn't take the time to really think of a better name.
A C-A-L in short directly translates to a **final-tree** and is quite easy to understand.
A C-A-L is a list of all the L-Nodes and N-Nodes with the corresponding number of leaf-nodes
that we need to attach to them.

For example, this C-A-L: `([3 ()])` could be translated in english to: 
"Attach 3 **final** leaf-nodes to the root-node".
Not that the nodes are represented by their AUP.
Actually, the example C-A-L above is a real C-A-L generated by the algorithm
when computing the trees with 8 nodes an 5 leaf-nodes.

It was computed for the first **internal-tree**:
```
1.
A
├── B
└── C
```
In order to convert this C-A-L to the corresponding **final-tree**,
we have to first create the **minimal-tree**.
The **minimal-tree** is the **internal-tree** but with **final** leaf-nodes
already attached to the L-nodes, we shouldn't forget it because the C-A-L
doesn't include these attachments.
Here is the **minimal-tree** of our **internal-tree**
```
tree:
A
├── B
│   └── 1
└── C
    └── 1
```
Here is the piece of code in charge of that in the `internal-tree-to-final-trees` function:
```clojure
	val minimum-tree = deep-copy(internal-tree)
	for node-dup in
		map(fn (n) : to-list $ in-reverse $ ascendant-unique-path $ n, leaf-nodes) do :
		add-deep-child(minimum-tree, node-dup, "1")
```
Note that we convert the AUPs to DUPs because it's easier for the `add-deep-child` function
to work with those.

Then we loop through each action of the C-A-L and we add the described number of children to the
descibed node.
This is because "attaching a **final** leaf-node" in practice is really the same as adding
a child to the node in question.

Our example C-A-L has only one action: attach 3 leaf-nodes to the root-node, so the
**final-tree** will look like that:
```
A
├── B
│   └── 1
├── C
│   └── 1
├── 2
├── 3
└── 4
```

Here is another C-A-L for the same **internal-tree**: `([1 (1L)] [2 (0L)])`
And here is the corresponding **final-tree**:
```
A
├── B
│   ├── 1
│   ├── 2
│   └── 3
└── C
    ├── 1
    └── 2
```
And as verification step, we notice that those **final-trees** have indeed
8 nodes and 5 leaf-nodes !

Here is the code in charge of applying the C-A-Ls in the `internal-tree-to-final-trees` function:
```clojure
	for al in C-A-L do:
		for i in 0 to al[0] do :
			add-deep-child(min-tree, to-list $ in-reverse $ al[1], to-string(i + 2))
	yield(min-tree)
```
 > Note in the code that I call it min-tree because it's just a modified version of the **minimal-tree**
but it really should be called final-tree.

This is how it works in a nutshell.

## Side note on optimization
When trying this implementation with big Ns and little Ms, there is an interesting problem
that brought my intention. All **internal-trees** are not valid, this because if the
**internal-tree** has more leaf-nodes than the number of final **leaf-nodes** (M), then it's
impossible to find any repartition of **final** leaf-node that will create a valid **final-tree**.

This problem becomes a huge issue when M stays small and N grows.
Because, what we are then asking the algorithm is to generate trees with
a great number of nodes but a little number leaf-nodes.
This intuitively means that the **final-trees** will look like long branches with a lot of nodes
but with a few **final** leaf-nodes.

Let's take the example of `N=10 and M=3`
Without any optimization, all the **internal-trees** are taken into consideration.
N - M = 7, so we look for **internal-trees** with 7 nodes. Here is a list of a
few **internal-trees** that won't work:
```
1.           2.          3.
R            R           R
└── R        ├── R       ├── R
    ├── 1    │   ├── 1   ├── R
    ├── 2    │   └── 2   └── R
    ├── 3    └── R           ├── 1
    ├── 4        ├── 1       ├── 2
    └── 5        └── 2       └── 3
```
These **internal-trees** all have 7 nodes but they all have way more than 3 L-nodes.

In total, there are 20 **internal-trees** that aren't suitable for this example.
I call those unsuitable **internal-trees** Bad Internal Trees, BAIT in short.
And the greater the N is, the exponentially worse it gets, here is a table that shows
how quickly the number BAITs grows:

| N  | M | Number of final-trees | Number of BAITs | Number of BAITS after optimization |
|----|---|-----------------------|-----------------|------------------------------------|
| 10 | 3 | 104                   | 20              | 0                                  |
| 11 | 3 | 160                   | 67              | 0                                  |
| 12 | 3 | 241                   | 207             | 0                                  |
| 13 | 3 | 345                   | 595             | 0                                  |
| 14 | 3 | 485                   | 1655            | 0                                  |
| 15 | 3 | 658                   | 4495            | 0                                  |

This is a really serious problem.
And the optimization I used so far doesn't seem to alter the number
of **final-trees** and reduced the number of BAITs to 0. If you find
any case where my optimization alters the number of **final-trees**
or misses a lot of BAITs (more than 0), then please open an issue.

The optimization is done in the `trees-with-n-nodes` function.
I corresponds to two optimization:
```
	val start-opti   = false
	val recurse-opti = false
```
These two switches allow us to quickly switch on and off the two optimizations.

The first switch corresponds to the start optimization.
```clojure
	val start = if start-opti :
		match(max-root-children) :
		(_:False) : N
		(s:Int) :
			if s >= 1 and s <= N :
				s
			else :
				N
	else :
		N
	for root-childs in in-reverse(1 to start) do :
```
This affects the root-childs count the function starts with.
Because, in reality, if we start with a root-child count greater than M, then
there is no way we can get a tree with less leaf-nodes than M.

The second optimization is just propagating this start-optimization to the recursive call
to the `trees-with-n-nodes` function in the `apply-repartition` function.
```clojure
	val m = if recurse-opti :
		match(max-root-children) :
		(_:False) : max-root-children
		(max-rc:Int) : max(1, max-rc - root-childs + 1)
	else :
		false
	for t in repartition apply-repartition(m):
		yield(t)
```

That's all for the optimization, and it seems to work fairly well
as the above table shows.


## Help
This is just a little project, but feel free to fork, change, extend or correct the code.

## License
This is delivered as it is under the well known MIT License.

**References that helped**
 - [lbstanza's learning book] : <http://lbstanza.org/chapter1.html>
 - [lbstanza's reference doc] : <http://lbstanza.org/reference.html>
 - [lbstanza's code examples] : <https://github.com/StanzaOrg/lbstanza/tree/master/examples>

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

   [lbstanza's learning book]: <http://lbstanza.org/chapter1.html>
   [lbstanza's reference doc]: <http://lbstanza.org/reference.html>
   [lbstanza's code examples]: <https://github.com/StanzaOrg/lbstanza/tree/master/examples>
