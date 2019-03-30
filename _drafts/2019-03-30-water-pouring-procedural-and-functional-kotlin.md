---
title: Water Pouring - Procedural and Functional Kotlin
layout: post
date: 2019/03/30 14:26:00
---

Let's try something different! In this post we are going to solve the famous "water pouring" problem using Kotlin two times: the procedural way and the functional way.

## The problem

In the standard "water pouring" puzzle you are given *n* glasses with different known capacities and you are asked to find a list of actions that leads to a desired quantity in one or more glasses.
You're only allowed to perform three kind of actions:

* Completely fill a glass
* Completely empty a glass
* Pouring one glass into another until the first one is empty or the second one is full

(I know what you're thinking... You can't cheat by filling "half" a glass... I can see you).

## The domain

Let's start by characterizing the domain. We are going to represent a state with the following data classes:

{% highlight kotlin %}
data class Glass(val capacity: Int, val value: Int)
data class State(val glasses: List<Glass>)
{% endhighlight %}

Next we define a move using an abstract class with an method that transforms the current state into the modified one:

{% highlight kotlin %}
sealed class Move {
    abstract fun update(state: State): State
}
{% endhighlight %}

The sealed class will have the three possible implementations:

<!--more-->

{% highlight kotlin %}
data class Fill(val index: Int): Move() {
    override fun update(state: State): State =
        State(state.glasses.mapAtIndex(index) { it.copy(value = it.capacity) })
}

data class Empty(val index: Int): Move() {
    override fun update(state: State): State =
        State(state.glasses.mapAtIndex(index) { it.copy(value = 0) })
}

data class Pour(val from: Int, val to: Int): Move() {
    override fun update(state: State): State {
        val fromGlass = state.glasses[from]
        val toGlass = state.glasses[to]

        val quantity = min(fromGlass.value, toGlass.capacity - toGlass.value)

        return State(
            state.glasses
                .mapAtIndex(from) { it.copy(value = it.value - quantity) }
                .mapAtIndex(to) { it.copy(value = it.value + quantity) }
        )
    }
}

// Utility extension function
private fun <T> List<T>.mapAtIndex(index: Int, f: (T) -> T) = 
    this.mapIndexed { i, t -> if (i == index) f(t) else t }
{% endhighlight %}

We'll also need a data structure to represent a list of moves which takes to a final state:

{% highlight kotlin %}
data class Path(val finalState: State, val moves: List<Move>) {
    fun extend(move: Move): Path {
        return copy(finalState = move.update(finalState), moves = moves + move)
    }
}
{% endhighlight %}

All this classes are immutable by design, and we'll exploit this in the following implementations.

## The procedural implementation

This problem can be represented as a graph, where each state is a node and each move is an edge connecting two neighbors state.
We are going to find the solution by exploring it using a Breath First Search algorithm.

Let's start by generating every possible move; we can fill every glass, empty every glass and pour every glass into a different one:

{% highlight kotlin %}
private fun generateMoves(): List<Move> {
    val result = mutableListOf<Move>()

    val indices = initialState.glasses.indices

    for (i in indices) {
        result.add(Move.Fill(i))
        result.add(Move.Empty(i))
    }

    for (i in indices) {
        for (j in indices) {
            if (i != j)
                result.add(Move.Pour(i, j))
        }
    }

    return result
}
{% endhighlight %}

We can now create a *solve* method, which takes a desired quantity and returns the shortest list of moves that define a solution.

We accumulate all the unexplored states into a *MutableList* which, at the beginning, is only containing the initial state.

Then we start iterating, at each step we:
* Pop the head from the frontier. If this state is a solution we are done. If the state is already explored we skip it.
* Add this state to the set of explored ones (this prevents any kind of cycling).
* Compute all the possible neighbors state by applying every possible move, and we add them to the end of the frontier.

This is guaranteed to complete, either we find a solution or we explore the complete search space.

{% highlight kotlin %}
fun solve(amount: Int): Path? {
    val explored = mutableSetOf<State>()
    val frontier = mutableListOf(Path(initialState, listOf()))

    while (frontier.isNotEmpty()) {
        val currentPath = frontier.removeAt(0)

        if (isSolution(amount, currentPath.finalState)) {
            return currentPath
        }

        if (explored.contains(currentPath.finalState)) {
            continue
        }

        explored.add(currentPath.finalState)

        for (move in moves) {
            val nextPath = currentPath.extend(move)
            frontier.add(nextPath)
        }
    }

    return null
}
{% endhighlight %}

## The functional implementation

Let's move to the function implementation. The overall structure will be similar, and we'll start by changing the *generateMoves* method:

{% highlight kotlin %}
private fun generateMoves(): List<Move> {
    val indices = initialState.glasses.indices

    val fillMoves = indices.map { Move.Fill(it) }
    val emptyMoves = indices.map { Move.Empty(it) }
    val pourMoves = indices
        .flatMap { i -> indices.map { j -> i to j } }
        .filter { (i, j) -> i != j }
        .map { (i, j) -> Move.Pour(i, j) }

    return fillMoves + emptyMoves + pourMoves
}
{% endhighlight %}

For the functional *solve* we are going to take inspiration from [Martin Odersky](https://www.coursera.org/learn/progfun2).

We define a recursive function *nextPaths* that given a list of paths with length *n* returns a sequence of lists of paths of length *n+1*.

This is done by calling the extend method on each of the input paths with all possible moves (filtering out explored states).

In this way we are guaranteed that shortest paths are processed first, leading to another implementation of the breath first search algorithm.

Also, Since Kotlin sequences are [lazily evaluated](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/-sequence/index.html), we know that longer paths are computed only if shortest paths are not solutions to the problem.

{% highlight kotlin %}
fun solve(amount: Int): Path? {
    return nextPaths(listOf(Path(initialState, listOf())), setOf())
        .flatten()
        .filter { isSolution(amount, it.finalState) }
        .firstOrNull()
}

private fun nextPaths(paths: List<Path>, explored: Set<State>): Sequence<List<Path>> {
    return if (paths.isEmpty()) emptySequence()
    else {
        sequence {
            val nextPaths = paths.flatMap { extendWithMoves(it, explored) }
            val nextExplored = explored + paths.map { it.finalState }

            yield(nextPaths)
            yieldAll(nextPaths(nextPaths, nextExplored))
        }
    }
}

private fun extendWithMoves(path: Path, explored: Set<State>): List<Path> {
    return moves.map { path.extend(it) }
        .filter { !explored.contains(it.finalState) }
}
{% endhighlight %}

## Conclusions

Let's try to highlight the takeaways of this short post:
* Both solutions work
* The procedural implementation was 2-3 times faster on the same input
* The functional implementation was more concise and arguably cleaner
* The functional implementation could be optimize to take advantages of multicores

You can check the full source code at: ???