Causal-Length Sets (CLSets) are a kind of set conflict-free replicated data type (CRDT), which are distributed data structures that guarantees their eventual convergence without locking. There are multiple Set CRDTs out there, ranging from naive but not so useful ones like the grow-only set (not so useful because you cannot remove from them), to Add-wins sets that fully tracks the causality (such that you can only remove elements that you observed) at the cost of keeping more metadata around.

CLSets are nice because they only require a single natural number (the causal-length) as the metadata, and allows both addition and removal operations. Based on the paper [*A Low-Cost Set CRDT Based on Causal Lengths*](https://dl.acm.org/doi/pdf/10.1145/3380787.3393678), here's the intuition about CLsets that "clicked" for me. At the risk of [explaining Monads like Burritos](https://byorgey.wordpress.com/2009/01/12/abstraction-intuition-and-the-monad-tutorial-fallacy/), I'll explain them in this post.

To understand how CLSets work, I think it makes sense to build up from simpler conflict-free sets.

(_An explanation of Grow-Only Set and 2P-set can be found in the paper [*A comprehensive study of CRDTs*](https://inria.hal.science/inria-00555588/document)_)

## Starting from Grow-Only Set

A grow-only set is a set that only allows adding elements. Since only addition is allowed, the set naturally grows in a single direction, and the *merge* operation can be defined as a set union. Set union naturally satisfies the associativity, commutativity, and idempotence requirements of state-based CRDTs, and is trivially a CRDT by itself.

## Adding removals with 2P set

A 2-phase-set (2P-set) builds upon this and adds another removal set to support removal operations. When elements are removed, they are added to this removal set, and the resulting set users see is the difference between the added set `S` and the removal set `R`.

This set is still not very useful in practice, because you can only remove an element once. Once it has been removed, further additions (concurrent or not) no longer has any effect on the resulting set.

## Adding more sets

If we want to allow adding elements after removal, a naive idea can be to add more sets to track if they have been re-added. Imagine a data structure (let's call this the *4P-set*) that keeps track of 4 different sets `(S1, R1, S2, R2)`, where the *merge* operation is defined as $(S_1 \setminus R_1) \cup (S_2 \setminus R_2)$. When adding an element, you first check if your element has been previously removed (whether it is in the `R1` set). If it is, then you want to reverse the removal by adding to `S2`, which takes precedence over the removals in `R1`. If not, then you can simply add the element in `S1`.

This construct allows adding and removing elements up to two times. Once you have removed the element a second time, which adds the element to `R2`, then you no longer have a way to add it back to the set.

What if we add even more sets, making this a *6P-set* or *8P-set*? That would get us closer to a practical set-semantics, but obviously it is not feasible to implement an *infinite-P set*.

## Noticing the equivalence

...or is it? The key thing to notice is that we have a lot of duplicates in our sets. If we add and remove the same element `e` over and over again, we are adding the same item to the sets `S1`, `R1`, `S2`, `R2`, ... Since the element `e` is the same in all of the sets, what if we somehow compress it?

You see where I am going? Instead of having an infinite number of sets, we can have one set that tracks all of the elements we have seen so far, and use a counter to keep track of whether it's in `S1` or `R1` etc. This can be implemented using a map in the format `{ e1: S1, e2: R1000 }`, where the values `S1` and `R1000` are just symbolic names of which "sets" the element is in. We only need to store one such name in the map, because we know that elements in `S2` implies that it must also be in `S1` and `R1`, so we only need to store the latest set that contains `e`.

If we map `S1` to 1, `R1` to 2, `S2` to 3, `R2` to 4, etc, then the value can be represented as a natural number, which the paper calls the *causal length*.

Personally, I like to think of the value as `(generation: int, removed: bool)` more, so it's obvious when elements are removed. When the pair is sorted lexicographically, and the `removed` boolean is sorted such that `true` > `false`, then it is equivalent to the causal length, while providing meaningful names for its semantics.

## Merging Semantics: Not really Add-Wins

One thing that I wondered was if _CLsets_ can be extended or modified to be truly Add-Wins, like an Observed-Removed Set. Unfortunately, this is not possible because we don't track the full causality in this set. Another node adding `e` is the same as us adding `e` into the set, and so if we add `e` into the set, it is possible for another node to remove it without having to ever observe our `add(e)` operation. The removal only need to see an `add(e)` from *any* node, in contrast to a traditional Add-Wins set which requires seeing the `add(e)` from *all* nodes that has executed the add operation.

While this can be problematic in some situations like using as the key-set for Maps, for Set semantics this fits reasonably well with the user's intentions. Imagine a to-do set (let's ignore ordering for now since we are talking about sets), if the user added an item to the set on node A, and then checks on node B that it is not there, and add it again on node B, they probably mean for that to be the same entry. If they later tries to remove the element (while A and B are still partitioned), they probably mean for the item to be removed on both nodes A and B. The key here is that in the to-do set, the elements are semantically equivalent, whereas in a Map key-set, the semantic meaning of the same key may be different, depending on the value associated with it.
