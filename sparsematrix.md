# SparseMatrixCSC for Graphs

This post describes the utility of the `SparseMatrixCSC` data structure as an adjacency matrix.
Readup: [Sparse Matrix](https://en.wikipedia.org/wiki/Sparse_matrix)


A graph can be represented using a `SparseMatrixCSC{Bool,Int}`, i.e. if `g[v,u] == true`, then there is an edge between vertices `u` and `v`.
Additional data, such as an index into a `DataFrame` can be stored instead of a mere `true` to indicate the presence of an edge.
Generating a random graph in this way is very easy:

```julia
V = 10^6
E = 10^7

# Directed graph
m = sprand(Bool,(V, V, E / (V * (V-1))   # Generate random matrix.
g = m - spdiagm(diag(m), 0)              # Remove self loops

# Undirected graph
h = g | g'
```

Here's why you should use `SparseMatrixCSC` over other representations, such vectors of vectors (`VoV`), edge lists, Dictionaries etc.

### Compact and Scalable
`SparseMatrixCSC` is very compact and probably is cache friendly, since most of the data is stored in contiguous memory locations. For example, creating a graph with `V` vertices, allocates `V` vectors. Creating a `SparseMatrixCSC` graph on the other hand allocates just three vectors.

### Linear Algebra support
`SparseMatrixCSC` comes with a ton of linear algebra routines. The `transpose` function for example, can be used to quickly reverse the graph. These routines also provide the scope for the implemention of [combinatorial graph algorithms](https://www.researchgate.net/profile/Viral_Shah7/publication/220093804_A_Unified_Framework_for_Numerical_and_Combinatorial_Computing/links/5415f7830cf2fa878ad3fbab.pdf).

### Subgraphs
Generating subgraphs becomes a lot easier with vertex-vector indexing. Eg:
```julia
# Subgraph containing first 10 vertices
g[1:10,1:10]
```


However, there were a few problems I faced:

### Listing a vertex's adjacencies
Listing adjacencies is quite tricky, and I couldn't find any established methods to do it quickly.
The simple way to do this is:
```julia
v = 1
adjacencies = g.rowval[nzrange(g, v)]
```
This is an expensive operation, since it involves a ranged indexing, which sets off a malloc. Another way to do this, is to use a "view" of the array.
```julia
v = 1
adjacencies = slice(g.rowval, nzrange(g, v))
```
This doesn't allocate as much memory, but still sets of a malloc. I didn't find any substantial speed improvement with this.
However, I've found a hacky way to get around allocating memory: Maintain an auxiliary array of size `V`, copy the adjacency list onto it, and resize the array.
```julia
adjvec = Vector{Int}(V)
v = 1
p1 = g.colptr[v]
p2 = g.colptr[v+1] - 1
resize!(adjvec, p2 - p1 + 1)
copy!(adjvec, 1, g.rowval, p1, p2 - p1 +1)
adjacencies = adjvec
```
This method is complex, but is substantially faster. It does cause problems if multiple processes are using the graph. However, one can get away with this by maintaining separate auxiliary arrays for each process.

### Storing metadata
The `SparseMatrixCSC` is designed to work with numeric data. Initially I experimented with storing edge metadata (numbers, strings, etc) in the adjacency matrix itself.
This strategy was messy and required a lot of coding. So I decided instead to use a `DataFrame` to store the metadata, and store only the index into the `DataFrame` in the
adjacency matrix. 
