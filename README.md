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
The most important function is this one:
```clojure
public defn trees-with-n-nodes (N: Int) :
    generate<Tree<String>> :
        for root-childs in in-reverse(1 to N) do :
            val tree = make-tree(root-childs)
            val left = N - root-childs - 1
            if left == 0 :
                yield(tree)
            else if left == 1 :
                set-child(tree, root-childs - 1, make-tree(1))
                yield(tree)
            else if left > 1 :
                val repartitions = repart(left, root-childs)
                for repartition in repartitions do :
                    for t in repartition apply-repartition:
                        yield(t)
```
`trees-with-n-nodes` generates all possible trees with N nodes.
For example, the `trees-with-n-nodes(4)` would generate those **final trees**
(don't mind the letters and the numbers, it's irrelevant):
```
1.       2.           3.            4.
R        R            R             R
├── 1    ├── 1        └── R         └── R
├── 2    └── R            ├── 1         └── 1
└── 3        └── 1        └── 2             └── 1
```
The first loop
`for root-childs in in-reverse(1 to N) do :`
iterates through all possible numbers of children that can have the root node.
On the last example, the values taken by root-childs would have been `3, 2 and 1`.
The maximum number of children the root node can have is `N-1`, and the minimum is 1.

```clojure
val tree = make-tree(root-childs)
```
For each iteration, we create a *base-tree* with the root-childs number of children on the root node.
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
val left = N - root-childs - 1
```
This next line associates to each *base-tree* a *left* counter which is the *left* number nodes that
we need to somehow add to the *base-tree* in order to get valid **final-tree**.
The first *base-tree* doesn't need any more nodes, it's *left* value is 0, the second needs one more node, it's *left* value is one, and the last one need 2 nodes, it's *left* value is 2.
`0, 1, 2`.

The next lines are quite easy to understand, they look at the *left* of the current *base-tree* and
act consequently.
```clojure
if left == 0 :
	yield(tree)
```
A *left* value of 0 means that the *base-tree* is already a **final tree**. So we can yield it.
This is the case for the first **final tree** of our example.

```clojure
else if left == 1 :
	set-child(tree, root-childs - 1, make-tree(1))
	yield(tree)
```
A *left* value of 1 means that the *base-tree* only needs 1 node to become a **final tree**.
By convention, we add the node to the last child-node of the *base-tree*.
This is the case for the second **final tree** of our example. 

```clojure
else if left > 1 :
	val repartitions = repart(left, root-childs)
	for repartition in repartitions do :
		for t in repartition apply-repartition:
			yield(t)
```
And here starts the interesting part. What should we do with a *left* value greather than 1.
Or in other words, what are all the possible ways to add 2, 3, 4, 5... to a tree.
We see that this solution include a "repart" function, let's check what it does.
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
Next we loop through each repartition, call this mysterious "apply-repartition" function
and loop through all trees generated by this function and generate them too. (I guess they would
use yield from in python)
```clojure
for repartition in repartitions do :
	for t in repartition apply-repartition:
		yield(t)
```

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
