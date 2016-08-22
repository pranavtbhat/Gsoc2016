---
layout: post
title: "GSoC 2016 Project: Graft.jl"
author: <a href="https://github.com/pranavtbhat">Pranav Thulasiram Bhat</a>
---

This blog post describes my work on [Graft.jl](https://github.com/pranavtbhat/Graft.jl), a general purpose graph analysis package for Julia. For those unfamiliar with graphs, or the need to analyze them, this would be a good [readup](https://www.cl.cam.ac.uk/teaching/1011/PrincComm/slides/graph_theory_1-11.pdf).

# Proposal
My proposal, titled [ParallelGraphs](https://github.com/pranavtbhat/Gsoc2016/blob/master/Proposal.md), was aimed at the developing a parallelized graph processing
library, aimed at large graph analysis. However, in the first month or so, we decided to work towards a more general framework that supports data analysis on
networks (graphs with attributes defined on vertices and edges).
Our change in direction was mainly motivated by:
- The problems associated with distributed graph computations. [This](http://www.frankmcsherry.org/graph/scalability/cost/2015/01/15/COST.html)
blog post was an eye opener.
- Only super-massive graphs, of the order of terabytes or petabytes, require distributed execution. Most useful graphs can be analyzed on a single compute node.
- Multi-threading is not yet fully established in Julia.
- [LightGraphs.jl](https://github.com/JuliaGraphs/LightGraphs.jl) already provides fast implementations for most sequential graph algorithms

So the modified proposal could be summarized as, the development of a package that supports:
- Vertex and edge metadata : Key value pairs for vertices and edges.
- Vertex labelling : Allow vertices to be referenced vertices, externally, through arbitrary Julia types.
- SQL like queries for edge data and metadata.
- Integration with `LightGraphs` : `LightGraphs` already provides solid implementations of graph algorithms.

# Graft
`ParallelGraphs` turned out to be a misnomer, since I was moving towards a more general purpose data analysis framework. So I chose the name `Graft`, a kind of abbreviation for Graph Toolkit. The following sections detail `Graft's` features:

## Vertex and Edge Metadata
Graphs are often representations of real world entities, and the information attached to them. Such entities (and their relationships), often have data attached to them.
While it's quite straightforward to store vertex data (a simple table will suffice), storing edges and their data is very tricky. The data should be structured on the
source and target vertices, should support random access and should be vectorized for queries.

At first I tried placing the edge data in a SparseMatrixCSC. This turned out to be a bad idea, because sparse matrices are designed for numeric storage.
A simpler solution is to store edge metadata in a DataFrame, and have a SparseMatrixCSC map edges onto indices for the DataFrame. This strategy needed a lot less
code, and the benchmarks were more promising. Mutations such as the addition or removal of vertices and edges become more complicated however.

## Vertex Labelling
Most graph libraries don't support vertex labelling. It can be very confusing to refer to a vertex by its (often long) integer identifier. It's also
computationally expensive to use non-integer labels in the implementation of the package (any such implementation would involve dictionaries). There is no reason, however,
for the user to have to use integer labels externally. Graft supports two modes of vertex labelling. By default, a vertex is identified by its internal identifier. A user
can assign labels of any arbitrary Julia type to identify vertices, overriding the internal identifiers. This strategy, I feel, makes a reasonable compromise between
user experience and performance.

## SQL Like Queries
Graft's query notation is borrowed from [Jplyer](https://github.com/davidagold/jplyr.jl). The `@query` macro is used to simplify the query syntax, and
accepts a pipeline of abstractions separated by the pipe operator `|>`. The stages are described through abstractions:

### `eachvertex`
Accepts a vertex expression, that is run over every vertex. Vertex properties can be expressed using the dot notation. Some reserved properties are `v.id`, `v.label`,
`v.adj`, `v.indegree` and `v.outdegree`.
Examples:

```julia
@query(g |> eachvertex(v.id == v.label)) |> all   # Check if the user has overridden the default labels
@query(g |> eachvertex(v.outdegree - v.indegree)) # Kirchoff's law :P
```

### `eachedge`
Accepts an edge expression, that is run over every edge. The symbol `s` is used to denote
the source vertex, and `t` is used to denote the target vertex in the edge. The symbol `e` is used to denote
the edge itself. Edge properties can be expressed through the dot notation. Some reserved properties are `e.source`, `e.target`, `e.mutualcount`, and `e.mutual`.
Examples:
```julia
@query g |> eachedge(e.p1 - s.p1 - t.p1)         # arithmetic expression on edge and constituent vertices' properties
@query g |> eachedge(s.outdegree == t.outdegree) # Checks if constituent vertices have the same outdegree
@query g |> eachedge(e.mutualcount)              # Counts the number of "mutual friends" between the participating vertices in each edge
```

### `filter`
Accepts vertex or edge expressions and computes subgraphs with a subset of vertices, or a subset
of edges, or both.
Examples:
```julia
@query g |> filter(v.p1 != v.p2)         # Remove vertices where property p1 equals property p2
@query g |> filter(e.source != e.target) # Remove self loops from the graph
```

### `select`
Returns a subgraph with a subset of vertex properties, or a subset of edge properties or both.
Examples:
```julia
@query g |> select(v.p1, v.p2) # preserve vertex properties p1, p2 and nothing else
@query g |> select(v.p1, e.p2) # preserve vertex property p1 and edge property p2
```

# Demonstration
The typical workflow I hope to support with `Graft` is:
- Load a graph from memory
- Use the query abstractions to construct new vertex/edge properties or obtain subgraphs.
- Run more complex queries on the subgraphs, or export data to `LightGraphs` and run computationally expensive algorithms there.
- Bring the data back into `Graft` as a new property, or use it to modify the graphs structure.

The following examples should demonstrate this workflow:

* [Baseball Players](https://github.com/pranavtbhat/Graft.jl/blob/master/examples/baseball.ipynb) demo: I spliced together two separate datasets, a table on baseball players
and a trust network. The resulting data is quite absurd, but does a good job of showing
the quantitative queries Graft can run.
* [Google+](https://github.com/pranavtbhat/Graft.jl/blob/master/examples/google%2B.ipynb) demo: This demo uses a real, somewhat large, dataset with plenty of text data.

# Future Work
- Graph IO : Support more graph file formats.
- Improve the query interface: The current pipelined macro based syntax isn't very easy to use, and the macro itself does some eval at runtime. I would like to move towards a cleaner composable syntax.
- New abstractions, such as Group-by, sort and table output.
- Database backends. A RDBMS can be used instead of the DataFrames. Or Graft can serve as a wrapper on a GraphDB such as Neo4j
- Integration with Com
More information can be found [here](https://github.com/pranavtbhat/Graft.jl/issues)

# Acknowledgements
This work was carried out as part of the Google Summer of Code program, under the guidance of mentors: [Viral B Shah](https://github.com/viralbshah) and [Shashi Gowda](https://github.com/shashi).
