# Gsoc2016

This repository describes my work as a GSoC student between May - August 2016. I worked on [Graft.jl](https://github.com/pranavtbhat/Graft.jl), a graph-data analysis tool for [Julia](http://julialang.org). 

## Mentors
* [Viral B Shah](https://github.com/viralbshah)
* [Shashi Gowda](https://github.com/shashi)

## Blog
* [Graft.jl](http://julialang.org/blog/2016/08/GSoC2016-Graft/)

## Quick Start
Install julia 0.5 from [here](http://julialang.org/downloads/). The Release candidates for 0.5 will do.

```julia
# Get Graft
julia> Pkg.update()
julia> Pkg.add("Graft")

# Load Package
julia> using Graft

# Load the sample graph
julia> g = samplegraph()

# Vertex data
julia> VertexDescriptor(g)
| VertexID │ Labels     │ age │ occupation    │
├──────────┼────────────┼─────┼───────────────┤
│ 1        │ "Abel"     │ 32  │ "Architect"   │
│ 2        │ "Bharath"  │ 35  │ "Banker"      │
│ 3        │ "Camila"   │ 21  │ "Chef"        │
│ 4        │ "Dalia"    │ 59  │ "Dentist"     │
│ 5        │ "Eduardo"  │ 24  │ "Economist"   │
│ 6        │ "Fabiola"  │ 37  │ "Florist"     │
│ 7        │ "Gaurav"   │ 46  │ "Geologist"   │
│ 8        │ "Hector"   │ 31  │ "Historian"   │
│ 9        │ "Ignacio"  │ 28  │ "Interpreter" │
│ 10       │ "Janardan" │ 59  │ "Judge"       │

# Edge data
julia> EdgeDescriptor(g)
│ Index │ Source     │ Target     │ relationship    │
├───────┼────────────┼────────────┼─────────────────┤
│ 1     │ "Abel"     │ "Bharath"  │ "father"        │
│ 2     │ "Abel"     │ "Camila"   │ "son"           │
│ 3     │ "Bharath"  │ "Abel"     │ "friend"        │
│ 4     │ "Bharath"  │ "Camila"   │ "friend"        │
│ 5     │ "Camila"   │ "Abel"     │ "familyfriend"  │
│ 6     │ "Camila"   │ "Bharath"  │ "familyfriend"  │
│ 7     │ "Camila"   │ "Dalia"    │ "wife"          │
│ 8     │ "Dalia"    │ "Camila"   │ "husband"       │
│ 9     │ "Dalia"    │ "Eduardo"  │ "friend"        │
│ 10    │ "Dalia"    │ "Fabiola"  │ "friend"        │
│ 11    │ "Dalia"    │ "Gaurav"   │ "neighbor"      │
│ 12    │ "Dalia"    │ "Hector"   │ "neighbor"      │
│ 13    │ "Dalia"    │ "Ignacio"  │ "teacher"       │
│ 14    │ "Dalia"    │ "Janardan" │ "student"       │
│ 15    │ "Eduardo"  │ "Dalia"    │ "father"        │
│ 16    │ "Eduardo"  │ "Fabiola"  │ "son"           │
│ 17    │ "Fabiola"  │ "Dalia"    │ "daughter"      │
│ 18    │ "Fabiola"  │ "Eduardo"  │ "mother"        │
│ 19    │ "Fabiola"  │ "Gaurav"   │ "aunt"          │
│ 20    │ "Gaurav"   │ "Dalia"    │ "nephew"        │
│ 21    │ "Gaurav"   │ "Fabiola"  │ "friend"        │
│ 22    │ "Hector"   │ "Dalia"    │ "friend"        │
│ 23    │ "Hector"   │ "Ignacio"  │ "friend"        │
│ 24    │ "Ignacio"  │ "Dalia"    │ "friend"        │
│ 25    │ "Ignacio"  │ "Hector"   │ "granddaughter" │
│ 26    │ "Ignacio"  │ "Janardan" │ "grandmother"   │
│ 27    │ "Janardan" │ "Dalia"    │ "sister"        │
│ 28    │ "Janardan" │ "Ignacio"  │ "sister"        │

# Some graph queries
julia> @query(g |> filter(v.age > 40)) |> VertexDescriptor
│ VertexID │ Labels     │ age │ occupation  │
├──────────┼────────────┼─────┼─────────────┤
│ 1        │ "Dalia"    │ 59  │ "Dentist"   │
│ 2        │ "Gaurav"   │ 46  │ "Geologist" │
│ 3        │ "Janardan" │ 59  │ "Judge"     │

julia> @query(g |> filter(e.relationship == "friend")) |> EdgeDescriptor
│ Index │ Source    │ Target    │ relationship │
├───────┼───────────┼───────────┼──────────────┤
│ 1     │ "Bharath" │ "Abel"    │ "friend"     │
│ 2     │ "Bharath" │ "Camila"  │ "friend"     │
│ 3     │ "Dalia"   │ "Eduardo" │ "friend"     │
│ 4     │ "Dalia"   │ "Fabiola" │ "friend"     │
│ 5     │ "Gaurav"  │ "Fabiola" │ "friend"     │
│ 6     │ "Hector"  │ "Dalia"   │ "friend"     │
│ 7     │ "Hector"  │ "Ignacio" │ "friend"     │
│ 8     │ "Ignacio" │ "Dalia"   │ "friend"     │
```

## Demonstration
* [Google+](https://github.com/pranavtbhat/Graft.jl/blob/master/examples/google+.ipynb)
* [Baseball Players](https://github.com/pranavtbhat/Graft.jl/blob/master/examples/baseball.ipynb)

## Contributions
My commits during the GSOC period:
- [Graft.jl](https://github.com/pranavtbhat/Graft.jl/commits/master?author=pranavtbhat)
- [LightGraphs.jl](https://github.com/JuliaGraphs/LightGraphs.jl/commits/master?author=pranavtbhat)
- [Julia](https://github.com/JuliaLang/julia/commits/master?author=pranavtbhat)

## Quick Links
* [Announcement](https://groups.google.com/forum/#!topic/julia-users/8wh-EzSEKxc) on the julia-users mailing list 
* [Package Build Status](http://pkg.julialang.org/?pkg=Graft#Graft)
* [License](https://raw.githubusercontent.com/pranavtbhat/Graft.jl/master/LICENSE.md)
* [Documentation](https://pranavtbhat.github.io/Graft.jl/stable)
