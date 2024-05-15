# Breaking the symmetry to create spin
In this part we will run a spin averaged DFT calculation, then we break the symmetry by applying a tensormoment and finally we will obtain the hybridization function from RSPt.  
The differences between LDA+$U$ and LSDA+$U$ are outlined in https://journals.aps.org/prb/abstract/10.1103/PhysRevB.97.184404.  
But for this Tutorial it is only important to know that it causes a double counting issues when calculating the spectra. Basically, the Impurity model will read in the exchange interaction from the local Hamiltonian in RSPt and then apply it again when it applies the Slater-parameters.
This can cause certain issues in the spectra for example an underprediction of the branching ratio in the XAS.


## Running a spin averaged DFT calculation
If you want to run a spin averaged DFT calculation, where you later on introduce the spin using tensormoments, you have to specify `spinpol` in the `symt.inp` file and you cannot specify `up` or `dn` in the `atom` block.
Instead use `a` and `b` to preserve the symmetry of the magnetic state.
````
atoms
 4
  0.00  0.00  0.00   28  l    b
  0.00  0.00  0.50   28  l    a
  0.50  0.50  0.25    8  l    a
  0.50  0.50  0.75    8  l    a

````
Next, to make sure that RSPt does not give these atoms a spin due to being lower in energy we average the two spin channels. This can be done by setting `prnt flag 11` in `dta/prnt_array` (before you make data) or in the `data` file from false
````
    .    1    .    2    .    3    .    4    .    5    .    6    .    7
fftffffftffffffffffftftfffffffftfffftffffffftffffffffffffffftfffffttffff
````
to true
````
    .    1    .    2    .    3    .    4    .    5    .    6    .    7
fftffffftftffffffffftftfffffffftfffftffffffftffffffffffffffftfffffttffff
````
## Breaking the symmetry
After running a few DFT calculations, we can introduce the spin in the form of the self-energy. This can be done by specifying the `tensmom` `Symbrk` keyword and the flag `0 1 1 1`, which corresponds to applying a tensormoment on the spin.
Then specify a magnitude at the end of the solver line of your cluster block. Here, positive numbers correspond to spin up and negative numbers to spin dn. The green.inp file to do this for NiO should look like this: 
````
! do not read any self-energy (if present)
inputoutput
F T

! start with symmetry-broken self-energy
! break the symmetry in the Sz channel
tensmom
Symbrk
0 1 1 1

! apply spin-independent double-counting correction
debug
Spin_avg_dc

cluster
1 eV UJ
1 2 1 1 0  7.2 0.9 Ewin -1.2 0.7
2  2  0.9 1.0
!          ^ this is a symmetry breaking magnitude. Positive sign is going to make spin moment pointing "up"

cluster
1 eV UJ
2 2 1 1 0  7.2 0.9 Ewin -1.2 0.7
2  2  0.9 -1.0
!          ^ this is going to make spin  moment pointing "dn"
````
You run only a single iteration with this `green.inp` file. After that you can run the calculation with your regular `green.inp` file until it is converged. 
Note, the spin will only show up in the Occupation part of the out file and nowhere else. If you have fullrel on you might see a small magnetic moment, which is the orbital moment.
## Getting the Hybridization function
Once your calculation is converged, you can extract the hybridization function from RSPt to do this simply specify the `spectrum` keyword `Hyb`, the `verbose` keyword `Dump` and the `debug` keyword `Dump`. 
````
debug
Dump

verbose
Dump

spectrum
Dos Pdos Hyb


````
Setting Hyb alone will ignore the offdiagonal elements in the hybridization function, which can contribute to the groundstate of the Hamiltonian and therefore to the shape of your spectra.
NiO is a cubic system and therefore, its natural basis is composed of $`e_g`$ and $`t_{2g}`$ orbitals. Therefore, it is recommended to use basis set `3`. You can do this in the cluster you use to apply the Hubbard $U$ or in an observer cluster.
````
cluster
1 
1 2 1 1 3 Ewin -1.2 0.7

cluster
1 eV UJ
1 2 1 1 0  8.0 0.9 Ewin -1.2 0.7
2  2  0.9

````
Here, you have to make sure to use the same energy window for the observer cluster as in the solver cluster.
When we look at the hybridization function on day 2, we will see that we will get sizeable offdiagonal elements in the hybridization function. This is because RSPt prefers a different z-axis due to the magnetic state breaking the cubic symmetry.
However, we know that the cubic symmetry describes our system very well. Therefore, we can simply align our z-axis for the decomposition with the z-axis of the cubic system using the Euler angles.
To do this rewrite your observer cluster to:
````
cluster
1 
1 2 1 1 3 Ewin -1.2 0.7 Euler 0 0 0 1
````
For the sake of practice, we want you to fit both hybridization functions and obtain similar results. To do this either run both calculations in separate folders or give the clusters a name using the Id keyword. 
For example Euler and Bare:
````
cluster
1 IdEuler
1 2 1 1 3 Ewin -1.2 0.7 Euler 0 0 0 1

cluster
1 IdBare
1 2 1 1 3 Ewin -1.2 0.7 
````
