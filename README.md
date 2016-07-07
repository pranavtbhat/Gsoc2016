# Gsoc2016
My project for Google Summer of Code 2016.

## General Performance Tips
- In general a low allocation count implies better performance. A very high allocation count usually indicates some form of type incompatibility or poor type handling. 
- The `@time` macro isn't very good at measuring performance, especially for small operations. Try `BenchmarkTools` instead.
- Use array comprehensions instead of map. Much faster, with far fewer allocations.
- `Sizehint!` to reduce allocation count.


## SparseMatrixCSC for Graphs

Readup: [Sparse Matrix](https://en.wikipedia.org/wiki/Sparse_matrix)

The `SparseMatrixCSC` is an excellent datastructure to store graphs. I've used this datastructure extensively in my package `ParallelGraphs`, to store both edge data and graph metadata. 

A graph can be represented using a `SparseMatrixCSC{Bool,Int}`, i.e. if `g[v,u] == true`, then there is an edge between vertices `u` and `v`. Generating a random graph in this way is very easy:

```julia
V = 10^6
E = 10^7

# Directed graph
m = sprandbool(V, V, E/V^2)   # Generate random matrix.
g = m - spdiagm(diag(m), 0)   # Remove self loops

# Undirected graph
h = g | g'
```

Here's why you should use `SparseMatrixCSC` over other representations, such vectors of vectors (`VoV`), edge lists, Dictionaries etc.

### Compact and Scalable
`SparseMatrixCSC` is very compact and probably is cache friendly, since most of the data is stored in contiguous memory locations. For example, creating a `VoV` graph with `V` vertices, allocates `V` vectors. Creating a `SparseMatrixCSC` graph on the other hand creates just three vectors.

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
However, I've found a hacky way to get around allocating memory: Maintain an auxillary array of size `V`, copy the adjacency list onto it, and resize the array.
```julia
adjvec = Vector{Int}(V)
v = 1
sz = g.colptr[v+1] - g.colptr[v]
resize!(adjvec, sz)
copy!(adjvec, 1, g.rowval, g.colptr[v], sz)
adjacencies = adjvec
```
This method is complex, but is substantially faster. It does cause problems if multiple processes are using the graph. However, one can get away with this by maintaining separate auxillary arrays for each process.

### Storing metadata using SparseMatrixCSC
`SparseMatrixCSC` was designed to operate on `Numbers`. However, one can store any arbitrary Julia object, as long as the method `zero` is defined on the type used. 
```julia
type VertexDescriptor
   id::Int
   name::ASCIIString
end
zero(::Type{VertexDescriptor}) = VertexDescriptor(0, "")

g = spzeros(VertexDescriptor, V, V)
g[1,2] = VertexDescriptor(1, "abc")
```
However, removing entries becomes a problem, as entries are removed only when they are set to numeric zero in the `Base` implementation. I've had to write my own methods to delete entries/entire columns.
