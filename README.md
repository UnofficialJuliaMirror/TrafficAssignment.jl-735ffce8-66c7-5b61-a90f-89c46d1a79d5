# TrafficAssignment.jl

[![TrafficAssignment](http://pkg.julialang.org/badges/TrafficAssignment_0.5.svg)](http://pkg.julialang.org/?pkg=TrafficAssignment)

[![Build Status](https://travis-ci.org/chkwon/TrafficAssignment.jl.svg?branch=master)](https://travis-ci.org/chkwon/TrafficAssignment.jl)
[![Build status](https://ci.appveyor.com/api/projects/status/8729wrjsv2rjga34?svg=true)](https://ci.appveyor.com/project/chkwon/trafficassignment-jl)
[![Coverage Status](https://coveralls.io/repos/chkwon/TrafficAssignment.jl/badge.svg)](https://coveralls.io/r/chkwon/TrafficAssignment.jl)



This package for [the Julia Language](http://www.julialang.org) does basically two tasks: (1) loading a network data and (2) finding a user equilibrium traffic pattern. See [Traffic Assignment](https://en.wikipedia.org/wiki/Route_assignment).

# Install


```julia
julia> Pkg.add("TrafficAssignment")
```

This will install [LightGraphs.jl](https://github.com/JuliaGraphs/LightGraphs.jl) and [Optim.jl](https://github.com/JuliaOpt/Optim.jl), if you don't have them already.

To check if works
```julia
julia> Pkg.test("TrafficAssignment")
```

# load_ta_network

This function loads a network data available in Hillel Bar-Gera's [Transportation Network Test Problems](http://www.bgu.ac.il/~bargera/tntp/)---it is now updated in [this github repository](https://github.com/bstabler/TransportationNetworks).

Example:
```julia
ta_data = load_ta_network("Sioux Falls")
# ta_data = load_ta_network("Anaheim")
# ta_data = load_ta_network("Barcelona")
# ta_data = load_ta_network("Chicago Sketch")
# ta_data = load_ta_network("Winnipeg")
```

The return value is of the TA_Data type, which is defined as
```julia
type TA_Data
    network_name::AbstractString

    number_of_zones::Int64
    number_of_nodes::Int64
    first_thru_node::Int64
    number_of_links::Int64

    start_node::Array
    end_node::Array
    capacity::Array
    link_length::Array
    free_flow_time::Array
    B::Array
    power::Array
    speed_limit::Array
    toll::Array
    link_type::Array

    total_od_flow::Float64

    travel_demand::Array
    od_pairs::Array

    toll_factor::Float64
    distance_factor::Float64

    best_objective::Float64
end
```

# ta_frank_wolfe

This function implements methods to find traffic equilibrium flows: currently, Frank-Wolfe (FW) method, Conjugate FW (CFW) method, and Bi-conjugate FW (BFW) method.

References:
- [Mitradjieva, M., & Lindberg, P. O. (2013). The Stiff Is Moving-Conjugate Direction Frank-Wolfe Methods with Applications to Traffic Assignment. *Transportation Science*, 47(2), 280-293.](http://pubsonline.informs.org/doi/abs/10.1287/trsc.1120.0409)

Example:
```julia
link_flow, link_travel_time, objective = ta_frank_wolfe(ta_data, log="off", tol=1e-2)
```

Available optional arguments:
* method=:fw / :cfw / :bfw (default=:bfw)
* step="exact" / "newton" : exact line search using golden section / a simple Newton-type step (default=:exact)
* log=:on / :off : displays information from each iteration or not (default=:off)
* max_iter_no=*integer value* : maximum number of iterations (default=2000)
* tol=*numeric value* : tolerance for the Average Excess Cost (AEC) (default=1e-3)

For example, one may do:
```julia
ta_data = load_ta_network("Sioux Falls")
link_flow, link_travel_time, objective = ta_frank_wolfe(ta_data, method=:cfw, max_iter_no=50000, step=:newton, log=:on, tol=1e-5)
```

The total system travel time can be simply computed as
```julia
system_travel_time = dot(link_travel_time, link_flow)
```

# Parallel Computing

During the all-or-nothing allocation procedure, this package supports parallel computing. If you want to start with two processes, use the following command to start Julia

```
julia -p 2
```

When you directly run your script, use the following command:

```
julia -p 2 your-script.jl
```



# Contributor
This package is written and maintained by [Changhyun Kwon](http://www.chkwon.net).
