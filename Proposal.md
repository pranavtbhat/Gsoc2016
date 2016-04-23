# GSoC2016 Proposal

## ParallelGraphs, A package for distributed and parallelized graph processing.

- Student: Pranav Thulasiram Bhat (pranavtbhat@gmail.com, [@pranavtbhat](https://github.com/pranavtbhat))
- Mentor : Shashi Gowda (shashigowda91@gmail.com, [@shashi](https://github.com/shashi))

## Summary
I hope to develop a package, ParallelGraphs, that will help analyse and manipulate massive graphs in a distributed environment. The package will adhere to two separate computing models; The first being the vertex centric Pregel model that relies on Bulk Synchronous Parallel infrastructure. The second model is a combinatorial approach that will involve matrix operations such as multiplication and vector indexing on distributed sparse matrices. ParallelGraphs will also provide flexibility in terms of the mode of computation offered. Compatibility with *LightGraphs* will ensure that smaller graphs can be dealt with using efficient sequential algorithms. The package will also experiment with CPU/GPU parallel algorithms with an aim of unifying all graph computation models into a single package.

## A Brief Introduction
Graphs are ubiquitous in their usage to represent real world objects and relations. As a consequence, data analysts are  often interested in the analysis and mining of graphs. Social networks for example contain vast amounts of information that can be mined and exploited. However, there is a catch. A large proportion of graph algorithms are nonlinear in terms of complexity, and the graphs themselves tend to be massive. 

The Mapreduce model is quite useful for facilitating the distributed execution of Data parallel tasks. However, the model struggles when it comes to graph algorithms. This is often due to the poor locality of computation in graph algorithms, and also due to the amount of data that needs to be moved between the map nodes and reduce nodes. A more useful programming model is the Bulk Synchronous Parallel model, which executes a series of synchronized super steps on several compute nodes. The BSP model relies heavily on message passing, and uses explicit synchronization barriers to ensure correctness in the execution of algorithms. ParallelGraphs will use the BSP model to support vertex centric algorithms such as Page Rank, Label Propagation etc.

Another approach to distributed graph processing, is to store graphs as distributed sparse matrices. The sparse matrix is well suited for graph representation as most useful graphs are sparse. Graph algorithms can then be implemented through matrix operations such as multiplication and transposition. Useful information can be derived from the graph through vector indexing and row/column aggregation. For example, Breadth First Search can be implemented through repeated vector multiplications on the sparse matrix. 

## A Case For Unifying Graph Libraries
There are generally three types of graph processing libraries:
- Single core sequential : The most common by far. The algorithms are executed sequentially on a single thread of computation. Ideal for smaller graphs. Examples include the Boost Graph Library, Networkx and LightGraphs.jl. 
- Parallelized : Graph algorithms are executed in parallel across CPU threads or GPU threads. Faster than sequential libraries, but cannot handle large graphs. Examples include the parallel extension to the Boost Graph Library and Gunrock.
- Multi node distributed : Graphs are partitioned and are distributed across computation nodes, and algorithms are run on each of these partitions. 

Depending on the type of the application involved, the optimal graph library to be used may vary. For example, a small graph with a few thousand vertices is best dealt with on a single computation node. Using a multi node library instead will incur significant communication and synchronization overheads, and would therefore be counterproductive. Whereas, the graphs used in social network analysis typically have billions of vertices. Such graphs cannot fit in the memory of a single node. A distributed environment is therefore a must to deal with such large graphs. Frank McSherry’s blog on the computational costs involved in achieving scalability in big data systems, [Scalability, but at what cost?] (http://www.frankmcsherry.org/graph/scalability/cost/2015/01/15/COST.html) presents this case.

Clearly it would be very convenient to data scientists to have a single graph processing package incorporating all three modes of computation.

## Why ParallelGraphs?
There are several big-data graph processing packages in existence. Apache Giraph, GraphX and Pegasus are some examples. Some of the issues I encountered while using these packages are:

- Several, if not all of these packages rely heavily on Hadoop.
- Most packages require input programs to be supplied through a jar file. Only graphX provided a command shell.
- None of the packages provided flexibility in terms of the mode of computation(sequential, parallel etc.)

ParllelGraphs will attempt to improve in these areas. The user will interact with the package through the Julia REPL, and will provide wrappers/interfaces to sequential and CPU/GPU parallel algorithms as well. Moreover, ParallelGraphs will add a key component to Julia's big-data analytics toolkit. As of now, there are no packages in Julia that support distributed or parallelized graph processing. 

## Implementation

Most algorithms will initially be implemented using two separate computation models:
- The vertex centric model Pregel model. This model adheres to the "think like a vertex" philosophy. Each algorithm consists of a vertex visitor function which is executed in parallel on each and every vertex. This model is mostly coupled with a Bulk Synchronous Parallel computation framework. ParallelGraphs will use ComputeFramework's dynamic and distributed task scheduler to implement its BSP infrastructure.  
- The combinatorial approach which will rely on the distributed sparse matrix implementation provided by ComputeFramework. Graph operations will be modeled through commonly used matrix operations such as multiplications, transposes and indexings.

ParallelGraph's distributed processing features will be implemented as follows:
- Flexible graph representations: ParallelGraphs will aim to support graph datastructures such as adjacency lists, adjacency matrices, sparse adjacency matrices, etc. It will also implement methods to convert these into the graph structures used at LightGraphs.jl. 
- Graph input: ParallelGraphs will support graph input formats such as *.dot*, *.gexf* and edgelists. I also hope to support reading and writing large graph datasets ,from distributed file systems such as HDFS, through *ComputeFramework*.
- Random Graphs: Graph generation will be supported through generators such as Erdős–Rényi, Watts and Strogatz and Power law. Parallelized graph generation kernels will be explored as well.
- Vertex Communication: ParallelGraphs will contain a message passing framework for vertex-to-vertex communication. This will be implemented using remote-calls and remote-references. 
- Reverse adjacency computation: This will be implemented as distributed matrix transpose computations for matrix representations. For adjacency list representations, message passing will be used to compute reverse adjacency lists.
- Vertex degree computation. Out-degrees can be evaluated easily on all graph representations, using row sums. The reverse adjacency list can be used to evaluate in-degrees.
- Connected components: ParallelGraphs will support two separate algorithms to isolate the connected components in a graph. The first method will use the vertex centric label propagation algorithm. Strongly connected components can be obtained by propagating labels through forward and backward edges. Weakly connected components can be obtained by propagating through forward edges only. The second and combinatorial approach will rely on repeated vector indexing on the parent vector. 
- Graph traversals BFS, DFS: ParallelGraphs will support both the vertex centric and the combinatorial approaches to graph traversals. The combinatorial approach will involve repeatedly multiplying a distance vector with the sparse matrix. 
- Shortest path algorithms: SSSP will be implemented as a specialized Breadth First Traversal in both the vertex centric and combinatorial approaches. All pair shortest path will be implemented by repeatedly multiplying a diagonal-distance matrix with the adjacency matrix.
- Pagerank algorithm: The page rank algorithm will be implemented using the vertex centric model.
- Max flow algorithms : I hope to implement the push relabel algorithm using the vertex centric model. 

ParallelGraphs will also attempt to support GPU parallel graph processing:
- GunRock is a Cuda library for GPU parallel graph computation. I hope to develop a wrapper over GunRock to allow users to access GunRock's algorithms from the Julia REPL. 
- Another possibility is to use *CUDArt.jl* to execute custom graph kernels.

Once Julia fully supports native threads, I hope to implement CPU parallel graph algorithms as well. While unifying all three models of computation (serial, parallel and distributed) is one of the goals of ParallelGraphs, its main goal is to provide highly efficient implementations of distributed graph algorithms that scale well. I hope to also develop a comprehensive benchmarking suite that conforms to the [Graph500 benchmark](http://www.graph500.org/specifications).  

## Demonstration
I have already started work on ParallelGraphs. So far I have implemented BFS and connected components through label propagation. My code can be found at: [ParallelGraphs.jl](https://github.com/pranavtbhat/ParallelGraphs.jl). You may need to checkout an earlier version of ComputeFramework however.

## Timeline
- May 23 - June 10 Rewrite package to use ComputeFramework scheduler. Design graph storage formats, and implement basic graph input.
- June 10 - June 20 Implement graph I/O and vertex to vertex message passing. 
- June 20 - July 1 Implement the BSP infrastructure and test for basic vertex visitors.
- July 1 - July 10 Implement graph traversals. Improve message passing, introduce message aggregation and filtering.
- July 10 - July 20 Implement visitor functions for distributed graph algorithms.
- July 20 - August 1 Implement combinatorial graph algorithms. Compare with vertex centric equivalents.
- August 1st - August 23 Performance optimizations and comprehensive testing. Explore CPU/GPU parallel graph algorithms. 

## About me
I'm a final year Computer Science student at the National Institute of Technology Karnataka, India. My academic interests lie primarily in Distributed and Parallel Computing, and Graph Theory. I joined the Julia community over a year ago, when I helped Rohit (@rohitvarkey) get started with Compose3D. Since then, I have contributed to the [flow algorithm module](https://github.com/pranavtbhat/LightGraphs.jl/tree/master/src/flow) in LightGraphs.jl and have started work on ParallelGraphs. I also volunteered at JuliaCon 2015, held in Bangalore.

My summer vacation begins in the first week of May. My masters education will most likely start in the last week of September, so I will be free for the entire duration of GSoc. I have no other commitments during this period, so I will be able to work on GSoC full time in Bangalore.

## References
- Leslie G. Valiant. 1990. A bridging model for parallel computation. Commun. ACM 33, 8 (August 1990), 103-111. [Bulk Synchronous Parallel](http://doi.acm.org/10.1145/79173.79181)
- Malewicz, Grzegorz, et al. "Pregel: a system for large-scale graph processing." Proceedings of the 2010 ACM SIGMOD International Conference on Management of data. ACM, 2010.
- Gilbert, John R., Viral B. Shah, and Steve Reinhardt. "[A unified framework for numerical and combinatorial computing](https://www.researchgate.net/profile/Viral_Shah7/publication/220093804_A_Unified_Framework_for_Numerical_and_Combinatorial_Computing/links/5415f7830cf2fa878ad3fbab.pdf)." Computing in Science & Engineering 10.2 (2008): 20-25.
- [Apache Giraph](https://github.com/apache/giraph)
- [Spark and GraphX](https://github.com/apache/spark)
- [ComputeFramework](https://github.com/shashi/ComputeFramework.jl)
- [LightGraphs](https://github.com/JuliaGraphs/LightGraphs.jl)
- [CUDArt.jl](https://github.com/JuliaGPU/CUDArt.jl)
- [Gunrock](https://github.com/gunrock/gunrock)


