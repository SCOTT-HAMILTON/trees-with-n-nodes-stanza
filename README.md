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
Following the previous example, the different *base-tree* would have looked like this:
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
This next line associates to each *base-tree* a *left* counter which is the *left* number of nodes that
we need to somehow add to the *base-tree* in order to get a valid **final-tree**.
The first *base-tree* doesn't need any more nodes, it's *left* value is 0, the second needs one more node, it's *left* value is one, and the last one need 2 nodes, it's *left* value is 2.
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

```clojure
else if left > 1 :
	val repartitions = repart(left, root-children)
	for repartition in repartitions do :
		for t in repartition apply-repartition:
			yield(t)
```
And here starts the interesting part. What should we do with a *left* value greather than 1 ?
Or in other words, what are all the possible ways to add 2, 3, 4, 5... nodes to a tree.
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
This repart function takes a first argument, the left value,
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
`((2 2) (1) (3)`. The order doesn't really matter at the end but we would like to keep it
the closest to the original repartition because the order corresponds to which child will become what.
So we would prefer the previous repartition to be grouped this way:
`((1) (2 2) (3)`.
Anyway, the reason behind grouping the repartition by value is that if you don't do that, you'll eventually end up generating **final trees** with the same topological structure which is wrong, we'll see an example of that later.

Now that we have groups, we loop though each group: 
```clojure
    for g in groups do :
        val r = g[0]
```
And we retrieve common "r" value of the group which is the number of nodes to add to each child.
This is better understood with an example, let's keep the same repartition as above `(1 2 2 3)`.
From this, we can deduce that the original *base-tree* looked like this (I named the root-children for clarity):
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
So you see that each grouped repartition value can be directly mapped to a child by reading it in depth-first order.

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
We make 2 different cases because the `trees-with-n-nodes` function doesn't work well on trees with 1 node.
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
This is exactly what the combinations-with-replacement function does.
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
Now that we have computed all combination possibilities for each group. We collect them together in a final list called `groups-ps`:
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
A2 and B2 are different possible trees with 2+1=3 nodes.
A3, B3 and C3 are different possible trees with 3+1=4 nodes.
```
And we're not done yet... Now make all possible combinations for all possibilities of each group.
This is done with the `all-combinations` function and the resulting combinations for our example look like that:
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
So we have:
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
So we give each child list to the apply-child-possibility function and we yield the returned tree.
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

Because this trees correspond to this unflattened combination:
`1.((A1) (B2 A2) (A3)) and 2.((A1) (A2 B2) (A3))`
Which means that that among all the generated combination possibilities of the group (2 2), there were the possibilities
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
