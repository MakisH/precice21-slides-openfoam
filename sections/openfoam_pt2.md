## Tutorials

<div class="container">
    <div class="col">
        <img src="images/openfoam/openfoam-openfoam_flat_plate_surface_T_poster_RBG.png" alt="CHT example: heated plate" style="margin-top:100px" /><br/>
        <img src="images/openfoam/openfoam-calculix_heat_exchanger_streamlines.png" alt="CHT example: heat exchanger" style="margin-top:100px"/>
    </div>
    <div class="col">
        <img src="images/openfoam/flap_perp.png" alt="FSI example: Perpendicular flap" /><br/>
        <img src="images/openfoam/cylinderFlap.png" alt="FSI example: Turek" />
    </div>
    <div class="col">
        <img src="images/openfoam/3DTube_scaled.png" alt="FSI example: 3D tube" style="margin-top:50px" />
        <img src="images/openfoam/pipe-pipe_rainbow_rotated.png" alt="FF example: pipe" style="margin-top:50px" />
    </div>
</div>

<small>Tutorials using OpenFOAM in our <a href="https://www.precice.org/tutorials.html">website</a>.</small>

---

# Some news

---

## Tricky: supporting multiple OpenFOAM versions


<img src="images/openfoam/branches.png" style="height:150px"/>

<small>Multiple Git branches, differing in small details</small>

vvv

## Option 1: Version-specific files

```c++
// Adapter.C

// Version-specific code with possible variants:
// - const_cast&lt;Time&&gt;(runTime_).setDeltaT(timestepSolver_, false);
// - const_cast&lt;Time&&gt;(runTime_).setDeltaTNoAdjust(timestepSolver_);
#include "version-specific/setDeltaT.H"
```

```bash
# Make/options

EXE_INC = \
    # ...
    -Ivariants/$(WM_PROJECT_VERSION) \
    # e.g. -Ivariants/v2012
```

<small>Read and comment in <a href="https://github.com/precice/openfoam-adapter/pull/128">PR #128</a>.</small>

vvv

## Option 1: Version-specific files

```
variants/
├── 4.0
│   └── version-specific
│       └── setDeltaT.H
├── 4.1 -> 4.0
├── 5.0
│   └── version-specific
│       └── setDeltaT.H -> ../../4.0/version-specific/setDeltaT.H
├── 6
│   └── version-specific
│       └── setDeltaT.H
├── 7
│   └── version-specific
│       └── setDeltaT.H -> ../../6/version-specific/setDeltaT.H
├── 8
│   └── version-specific
│       └── (many files...)
|
├── v1712
│   └── version-specific
│       └── setDeltaT.H -> ../../4.0/version-specific/setDeltaT.H
├── v1806 -> v1712
├── v1812 -> v1806
├── v1906 -> v1812
├── v1912 -> v1906
├── v2006 -> v1912
└── v2012 -> v2006
```

Could work, right?

vvv

## Option 1: Version-specific files

```
variants/
├── 4.0
│   └── version-specific
│       ├── directory_type.H
│       ├── init.H
│       └── setDeltaT.H
├── 4.1 -> 4.0
├── 5.0
│   └── version-specific
│       ├── directory_type.H -> ../../4.0/version-specific/directory_type.H
│       ├── init.H
│       └── setDeltaT.H -> ../../4.0/version-specific/setDeltaT.H
├── 6
│   └── version-specific
│       ├── directory_type.H -> ../../5.0/version-specific/directory_type.H
│       ├── init.H
│       └── setDeltaT.H
├── 7
│   └── version-specific
│       ├── directory_type.H
│       ├── init.H
│       └── setDeltaT.H -> ../../6/version-specific/setDeltaT.H
├── 8
│   └── version-specific
│       ├── directory_type.H -> ../../7/version-specific/
│       ├── init.H -> ../../7/version-specific/
│       ├── setDeltaT.H -> ../../7/version-specific/
│       ├── CHT/HeatFlux.C
│       ├── CHT/HeatTransferCoefficient.c
│       ├── FSI/Force.C
│       ├── FSI/Force.H
│       ├── FSI/Stress.H
│       └── ...
|
├── default -> 5.0
├── dev -> 7
├── v1712
│   └── version-specific
│       ├── directory_type.H -> ../../4.0/version-specific/directory_type.H
│       ├── init.H
│       └── setDeltaT.H -> ../../4.0/version-specific/setDeltaT.H
├── v1806 -> v1712
├── v1812 -> v1806
├── v1906 -> v1812
├── v1912 -> v1906
├── v2006 -> v1912
└── v2012 -> v2006
```

Not future-proof (at least not for openfoam.org)

vvv

## Option 2: Delegate to OpenFOAM
<small>Proposal for CHT by <a href="https://github.com/TEFEdotCC">Thomas Enzinger</a> in <a href="https://github.com/precice/openfoam-adapter/pull/150">PR #150</a></small>


Move wall treatment to a boundary condition
  - Boundary condition provided, compiled separately
  - Similar need for a few `#ifdef`
  - More "OpenFOAM-native" design

How general do we want to be?


---

## New: Pressure-based solver type selection

```c++
dimensionSet pressureDimensionsCompressible(1, -1, -2, 0, 0, 0, 0);
dimensionSet pressureDimensionsIncompressible(0, 2, -2, 0, 0, 0, 0);

if (mesh_.foundObject&lt;volScalarField&gt;("p"))
{
  volScalarField p_ = mesh_.lookupObject&lt;volScalarField&gt;("p");

  if (p_.dimensions() == pressureDimensionsCompressible)
    solverType = "compressible";
  else if (p_.dimensions() == pressureDimensionsIncompressible)
    solverType = "incompressible";
}
```

<small>Thanks to David Schneider (TUM) for <a href="https://github.com/precice/openfoam-adapter/pull/124">adding this</a> (Merged: March 2020).</small>

---

## New: Write stresses (FSI)

- Before: write forces, read displacements
- Now: write forces **or stresses**, read displacements
    - No need for conservative mapping!

<small>Thanks to David Schneider (TUM) for <a href="https://github.com/precice/openfoam-adapter/pull/125">adding this</a> (Merged: June 2020).</small>

---

## Improved: 2D mode

- Before:
    - FSI: **point**-displacements over face-nodes and forces over face-centers
    - 2D: two layers of face-nodes &rarr; RBF mapping 💥
- Now:
    - 2D: read **cell**-displacements on face-centers<br/> &rarr; RBF mapping 😌

<small>Thanks to David Schneider (TUM) for <a href="https://github.com/precice/openfoam-adapter/pull/147">adding this</a> (Merged: December 2020).</small>

---

## Future: Unit & integration tests

- Already CI with system tests
- Wish: Test specific parts of the adapter
    - Unit tests with [Catch2](https://github.com/catchorg/Catch2)
    - Integration tests with [Google Test](https://github.com/google/googletest)
    - Other ideas?

<small>Prototype <a href="https://github.com/precice/openfoam-adapter/pull/122">contributed</a> by Qunsheng Huang (TUM).</small>