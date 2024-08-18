# xtal_lego
Crystal Generator with the desired local coordination environment. 


### Initialization
```python
from pyxtal import pyxtal
from xtal_lego.builder import mof_builder

# Set reference graphite sp2 structure
xtal = pyxtal()
xtal.from_spg_wps_rep(194, ['2c', '2b'], [2.46, 6.70], ['C', 'C'])
cif_file = xtal.to_pymatgen()

builder = mof_builder(['C'], [1], db_file='mof_sp2.db')
builder.set_descriptor_calculator(mykwargs={'rcut': 2.0})
builder.set_reference_enviroments(cif_file)
builder.set_criteria(CN={'C': [3]})
print(builder)
```

```
------MOF Builder------
System: C1
Database: mof_sp2.db
Log_file: mof.log
Descriptor: SO3 descriptor with Cutoff:  1.800 lmax: 4, nmax: 2, alpha: 1.500
neighborlist: ase
Reference enviroments (1, 15)
Criterion_CN: {'C': 3}
Criterion_cutoff: None
Criterion_Dimension: 3
Criterion_exclude_ii: False
```

The above setup include the following important information:

- `Reference enviroment`: the reference atomic environment based on the input crystal
- `Calculator`: the way to compute local atomic environment
- `database file`: a database file to store the structures generated by this instance
- `log file`: a log file to store information all activities

### Optimization of a single crystal    
After the initialization, the `mof_builder` class can take an input crystal and then perform optimization to get the crystal with the target reference environment

```python
# Generate a diamond structure and optimize its similarity
xtal0 = pyxtal()
xtal0.from_spg_wps_rep(227, ['8a'], [3.6], ['C'])
xtal0, sim = builder.optimize_xtal(xtal0)
print(xtal0.get_xtal_string(dicts={"Sim": float(sim)}, header='After Opt'))

# Try to optimize at the subgroup representation 227->210
sub = xtal0.subgroup_once(H=210)
sub0, sim = builder.optimize_xtal(sub)
print(sub0.get_xtal_string(dicts={"Sim": float(sim)}, header='After Opt'))

# Try to optimize at the subgroup representation 227->210->212
sub = sub.subgroup_once(H=212, group_type='t+k')
sub0, sim = builder.optimize_xtal(sub)
print(sub0.get_xtal_string(dicts={"Sim": float(sim)}, header='After Opt'))
sub0.to_file('opt.cif')
```

```
After Opt*  8   1  227 Fd-3m          4.03    13.386 8a
After Opt*  8   1  210 F4132          4.03    13.386 8b
After Opt*  8   2  212 P4332          2.46     0.000 8c
   0*  8   2  212 P4332          2.46  False        13.037 =>  0.000   8c
```

This example does not only show how to call `optmize_xtal` function via `mof_builder`, 
but also illustrate that optimization under different space group representation can lead to different results.

### Post-analysis 

You may need to run two external packages 
- `GULP`: based on classical force field
- `dftbplus`: based on density functional tight-binding

Detailed instructions can be found at the subdiectory of [Installation](https://github.com/qzhu2017/MOF-Builder/tree/main/Installation)


