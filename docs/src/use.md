# Usage

To calculate potential energy surface refer to [PotentialCalculation](https://github.com/tjjarvinen/PotentialCalculation.jl).Ones you have potential energy calculated you can open
it for fitting by using

```@example 1
using PotentialFitting

# There is an example potential in test/data directory
using PotentialDB
r = defaultregistry()
data=loadpotential(r,"4")
```

Potential can be viewed by energy

```@example 1
# Collumn 1 from data
plot_potential(data,1)
```

or ineractively

```julia
plot_potential(data)
```

and by geometry with external program (VMD here)

```julia
visualize_points(data["Points"]; command="vmd")
```

!!! note "Note"
      This will fail when using IJulia. Due to IJulia closing external programs.

## Setting up Molecules

Next part in defining topology for the potential. This is started by creating two
molecules. The information is in the loaded file.

```@repl 1
m1, m2 = get_molecules(data)
```

If needed atoms can be flagged as identical.

```@example 1
# Atoms 2 and 3 are identical
makeidentical!(m1, (2,3))
```

## Potential Topology

Next we need to define topology for the potential.

```@example 1
mpp = MoleculePairPotential(m1,m2, LennardJones())
```

### Finetuning Potential

Alternatively potential can be tuned completely by adding potentials one by one.

```@example 1
# Array where topology is saved
topo=[]

#We can push potential to to this array one at the time
push!(topo,
      # Molecule 1 has 5 atoms so index 6 is molecule 2, or argon now
      PairPotentialTopology(LennardJones(), 1,6)
     )
nothing # hide
```


If needed we can specify which atoms should be treated as identical, by adding
information for it  in the topology.

```@example 1
# Atoms 2 and 3 of molecule 1 have same potential to to atom 1 of molecule 2
push!(
      topo,
      PairPotentialTopology(LennardJones(), [(2,6), (3,6)])
)
nothing # hide
```


If default form of potential is not enough it can be tuned, by giving it as an input.

```@example 1
push!(
      topo,
      PairPotentialTopology(GeneralPowers(-6,-12), 4,6)
)
push!(
     topo,
     PairPotentialTopology(GeneralPowers(-6,-8, -10, -12), 5,6)
)
nothing # hide
```

Here we used general polynomial potential ```GeneralPowers``` to make customized
polynomic potential.

We can now create potential.

```@example 1
mpp1=MoleculePairPotential(m1,m2)
mpp1.topology = topo

show(mpp1)
```

## Preparing Data for Fitting

To do fitting itself we need to prepare fit data.

```@example 1
fdata = FitData(mpp, data["Points"], data["Energy"])

# Also this works and can be used add data from different sources
# fdata = FitData(mpp, data1, data2,...)

nothing # hide
```

At this point we can add weights to data.

```@example 1
# If energy is more than 1500 cm⁻¹ weigth is zero
setweight_e_more!(fdata, 0, 1500)

# If energy is less than 80 cm⁻¹ weigth is 4
setweight_e_less!(fdata,4,80)
nothing # hide
```

## Fitting Potential

We also need to create fitting model. At the current moment only linear models
can be used. Here we take normal linear regression, but any linear model
supported by [ScikitLearn](https://github.com/cstjean/ScikitLearn.jl/)
can be used.

```@example 1
using ScikitLearn
@sk_import linear_model: LinearRegression

model = LinearRegression()
nothing  # hide
```



To do fitting itself.

```@example 1
fit_potential!(model, mpp, fdata)
```

## Inspecting Fitted Potential

You can inspect the fit by calculating RMSD.

```@example 1
# Unit is hartree
rmsd(data["Points"], data["Energy"], mpp)
```



Alternatively you can visualize the fit with various methods.

```@example 1
plot_compare(data["Points"][:,1], data["Energy"][:,1], mpp,
             leg=true, size=(600,300))
```



For more visualizations take a look for
- [`plot_compare`](@ref)
- [`scan_compare`](@ref)
- [`scan_vizualize`](@ref)
- [`visualize_points`](@ref)
