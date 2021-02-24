## What does the adapter do?

<img src="images/openfoam/openfoam_adapter_overview_linking.svg" />

vvv

## What does the adapter do?

<img src="images/openfoam/openfoam_adapter_overview_data.svg" />

vvv

## What does the adapter do?

<img src="images/openfoam/openfoam_adapter_overview_checkpointing.svg" />

vvv

## What does the adapter do?

<img src="images/openfoam/openfoam_adapter_overview_timestep.svg" />

---

## Configuration

<img src="images/openfoam/config.svg" />

vvv

## Main file: `system/preciceDict`

```c++
preciceConfig "precice-config.xml";

participant Fluid;

modules (FSI);

interfaces
{
  Interface1
  {
    mesh              Fluid-Mesh-Faces;
    patches           (flap);
    locations         faceCenters;
    
    readData
    (
    );
    
    writeData
    (
      Forces0
    );
  };
  
  Interface2
  {
    mesh              Fluid-Mesh-Nodes;
    patches           (flap);
    locations         faceNodes;
    
    readData
    (
      Displacements0
    );
    
    writeData
    (
    );
  };
};

FSI
{
  rho rho [1 -3 0 0 0 0 0] 1;
}
```

<small>No need for yaml-cpp anymore! (since February 2020)</small>


