# Calculate the spectra
After fitting the hybridization function and closing the last window that `finiteH0.py` opens, the total number of bath states is printed to the terminal and the files h0.pickle abd h0.dict are created.
Now, we can go on to obtain the spectra. for this let us look at the joscript `script_Xbath.sh`. It should look like this
````
#!/bin/bash
#SBATCH -N 1
#SBATCH -t 01:00:00
#SBATCH -A naiss2023-1-44 
#SBATCH -J NiO-calc
# 
n0imp=8
doccs=8
nBath3d=20
# Number of valence bath states coupled to 3d-orbitals.
nBath3dval=20
ct=2.50
delta=0.4
delta_2=0.1
POS=5.5
#run RIXS?
deltaRIXS=-0.05
#deltaRIXS=-1.0
deltaNIXS=-1.0
# Filename of the non-relativistic non-interacting Hamiltonian
h0_filename="h0.pickle"
# Filename of the radial part of the correlated orbitals.
radial_filename="radial.data"

#echo "Number of MPI ranks to use: $ranks"
echo "Number of computational nodes: $SLURM_JOB_NUM_NODES"  > impurity.log
echo "Impurity occupation: $n0imp" > impurity.log
echo "Total number of bath states: $nBath3d" >> impurity.log
echo "Number of valence bath states: $nBath3dval" >> impurity.log
echo "H0 filename: $h0_filename" >> impurity.log
echo "Radial wavefunction filename: $radial_filename" >> impurity.log
echo "Charge transfer: $ct" >> impurity.log
echo "Run RIXS: (deltaRIXS >0) $deltaRIXS " >> impurity.log

mpprun python -m impurityModel.ed.get_spectra $h0_filename $radial_filename --Fdd 7.5 0 9.9  0 6.6 --Fpd 8.9 0 6.8 --Gpd 0 5.0 0 2.8 --xi_2p 11.63 --xi_3d 0.096 --VP 0.0 --n0imps 6 $n0imp --nBaths 0 $nBath3d --nValBaths 0 $nBath3dval --dnConBaths 0 0 --chargeTransferCorrection $ct --deltaRIXS $deltaRIXS --deltaNIXS $deltaNIXS --delta $delta --nPsiMax 6 --delta_2 $delta_2 --POS $POS --doccs $doccs >> impurity.log
````
Now, let us go through every part that is set here.  
-`n0imp` (integer): The nominal occupation of the orbital you wish to excite into.  
-`doccs` (float): Occupation of the said Orbital for double counting (Nominal works well, but sometimes it is worth trying the occupation RSPt gives you).  
-`nBath3d` (integer): Total number of bath states for the Orbitals you want to excite into. finiteH0.py prints it at the end of the run.  
-`nBath3dval` (integer): Number of valence bath states for the Orbitals you want to excite into.  
-`ct` (float): Double counting correction, physically corresponds to  
-`delta` (float): lorentizian broadening of the entire spectrum due to lifetime effects  
-`delta_2` (float): additional broadening of the $L_2$-edge due to the super Coster-Kronig effect.  
-`POS` (float): Energy after which the extra broadening is applied aim to have it in a region were the intensity between the $L_3$ and $L_2$ edge is zero or at least small.  
-`deltaRIXS`(float): broadening for the energyloss part in RIXS, negative values make the program skip the calculation of the RIXS spectrum.   
-`deltaNIXS`(float): broadening for the energyloss part in NIXS, negative values make the program skip the calculation of the NIXS spectrum.   
-`Fdd` (tuple): $`F_{dd}^k$` Slater parameters from RSPt in order of $k$. Note that $`F_{dd}^k`$ is 0 when k is odd.  
`Fpd` (tuple): $`F_{pd}^k$` Slater parameters from RSPt in order of $k$. Note that $`F_{pd}^k`$ is 0 when k is odd.   
-`Gdd` (tuple): $`G_{pd}^k$` Slater parameters from RSPt in order of $k$. Note that $`G_{pd}^k`$ is 0 when k is even.  
-`xi_2p` (float): 2 $p$ spin-orbit coupling in eV. You can calculate it from the energy difference of the core states in out. The difference is then multiplied by $`-\frac{2}{3}`$.  
-`xi_3d` (float): 3 $d$ spin-orbit coupling in eV.  
-`VP` (float): shift of the 2 $p$ energy level in eV. Only important if you have multiple different sides contributing to the same absorption edges. The energy differences between the core states in RSPt give a good indication of this value.  
-`dnConBaths` (Tuple): number of states to consider in the conduction band. Put the second number to 1 or 2 if you include bath states in the conduction band. Otherwise both should be 0.  
-`nPsiMax` (integer): number of wavefunctions to calculate to determine the thermal ground state. It should be larger than the number of bath states considered for making the spectra in impurity.log. As you do not know if you used enough otherwise.  
Now that we have to set the parameters according to the Slaterparameters we determined on the first day. We can simply run the script using
````
sbatch script_Xbath.sh
````
or getting an interactive node and running it via
````
./script_Xbath.sh
````
After the calculation is done, it has produced the different spectra files and a file called impurity.log. 
First let us open a spectra file for example `XAS.dat` using a texteditor. The top od the file should look like
````
# E  sum  T1  T2  T3 ...
-25.0000   0.0008   0.0003   0.0003   0.0003
-24.9499   0.0008   0.0003   0.0003   0.0003
-24.8999   0.0008   0.0003   0.0003   0.0003
-24.8498   0.0008   0.0003   0.0003   0.0003
-24.7998   0.0008   0.0003   0.0003   0.0003
-24.7497   0.0008   0.0003   0.0003   0.0003
````
Here, the first column is the excitation energy in eV. The second one is the sum of all the following columns and therefore contains the XAS signal.
The last three are the contributions of the XAS in just the `x,y` or `z` direction. The `z` direction is included to take the random orientation in multicrystaline samples into account. You can just remove it if you calculate perfect crystals. Which is set in your code folder in `impurityModel/ed/get_spectra.py` under the term epsilons.
It is currently set to `epsilons = [[1, 0, 0], [0, 1, 0], [0, 0, 1]]` if you want to calculate the XMCD then set
````
epsilons = [[ -1/np.sqrt(2), -1j/np.sqrt(2), 0], [ 1/np.sqrt(2), -1j/np.sqrt(2), 0],[0,0,1]]
````
Now, the XAS will still be on the second column of the XAS.dat as we add the contributions of left and right circularly polarized light. To get the XMCD just calculate the difference between columns 3 and 4.
