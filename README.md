# Gsoc2016
My project for Google Summer of Code 2016.

## Mentors
* [Viral B Shah](https://github.com/viralbshah)
* [Shashi Gowda](https://github.com/shashi)


## Project Proposal

You can find my proposal for Gsoc2016 [here](https://summerofcode.withgoogle.com/serve/6003629811040256/)

The proposal describes plans to develop a parallelized graph processing library, aimed at large graph analysis.
However, in the first month or so, we decided to work towards a package that supports graph metadata,
vertex labelling, and SQL like queries for graphs.
Our change in direction was mainly motivated by:

- The problems associated with distributed graph computations. [This](http://www.frankmcsherry.org/graph/scalability/cost/2015/01/15/COST.html)
blog post was an eye opener.
- Only super-massive graphs, of the order of terabytes or petabytes, require distributed execution. Most useful graphs can be analyzed on a single node.
- Multi-threading is not yet fully established in Julia.
- [LightGraphs.jl](https://github.com/JuliaGraphs/LightGraphs.jl) already provides fast implementations for most sequential graph algorithms
We therefore felt that it would be more productive to develop a general purpose graph framework, that allows users to mine data from graphs.


## Graft
Most of my work during the coding period has been devoted to the development of [Graft.jl](https://github.com/pranavtbhat/Graft.jl), a general purpose
graph framework. Graft provides the following features:
- Vertex and edge metadata: Key value pairs can be attached to vertices and edges. Graft uses tables from [DataFrames.jl](https://github.com/JuliaStats/DataFrames.jl)
to store metadata.
- Vertex labelling: By default, a vertex is identified by its internal integer identifier. A user can assign labels of any arbitrary Julia type to identify vertices. However, these labels can be used only externally, since vertex label resolution can impose a significant overhead.
- Graph queries: Graft provides a pipelined query macro system, that allows users to construct complex sql-like queries on graph metadata, and structure.

## Demo
* [Google+](https://github.com/pranavtbhat/Graft.jl/blob/master/examples/google+.ipynb)
* [Baseball Players](https://github.com/pranavtbhat/Graft.jl/blob/master/examples/baseball.ipynb)

## Contributions
My commits during the GSOC period:
- [Graft.jl](https://github.com/pranavtbhat/Graft.jl/commits/master?author=pranavtbhat)
- [LightGraphs.jl](https://github.com/JuliaGraphs/LightGraphs.jl/commits/master?author=pranavtbhat)
- [Julia](https://github.com/JuliaLang/julia/commits/master?author=pranavtbhat)
