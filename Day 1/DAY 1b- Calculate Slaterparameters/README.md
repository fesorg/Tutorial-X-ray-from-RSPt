# Calculation of the higher-order Slater-parameters

## Setting up the Calculation
After the calculation is converged create a copy of it. In this copied folder we will modify the `data` file in a way that we place the 2$`p`$ electrons in the semi-core state and the 3$`p`$ in the core.
This is done by changing the occ part of the `element` file or the element part of the `data` from:
````
     Z    nc      Sinf/S     . Sws/S  ....     ScoFlag
   28.     7          1.    1.    1.  10.0           p
     n     k         occ        flag
     1    -1          2.           0
     2    -1          2.           0
     2     1          2.           0
     2    -2          4.           0
     3    -1          .0           0
     3     1          .0           0
     3    -2          .0           0

````
 to
````
     Z    nc      Sinf/S     . Sws/S  ....     ScoFlag
   28.     7          1.    1.    1.  10.0           p
     n     k         occ        flag
     1    -1          2.           0
     2    -1          2.           0
     2     1          .0           0
     2    -2          .0           0
     3    -1          .0           0
     3     1          2.           0
     3    -2          4.           0
````
and the Basis block in the same file from
````
    12 Bases
     0     1     1
     0     1     2
     0     1     3
     1     1     1
     1     1     2
     1     1     3
     2     1     1
     2     1     2
     0     2     1
     0     2     2
     1     2     1
     1     2     2
     4     4     3     4     5     6     7     8     9
     0     0    -1     0     0     0     0     0     0
     3     3     3     4     5     6     7     8     9
    -1    -1     0     0     0     0     0     0     0
````
to:
````
12 Bases
     0     1     1
     0     1     2
     0     1     3
     1     1     1
     1     1     2
     1     1     3
     2     1     1
     2     1     2
     0     2     1
     0     2     2
     1     2     1
     1     2     2
     4     4     3     4     5     6     7     8     9
     0     0    -1     0     0     0     0     0     0
     3     2     4     4     5     6     7     8     9
    -1    -1    -1     0     0     0     0     0     0

````
Note that we put the 4$`d`$ electrons into `energy set 2` and added the `-1` flag. This is done so that `energy set` 1 and 2 are orthogonal and the electrons can not move between them, which would give us the wrong interaction.
After we modified the `data` file we now have to modify the `green.inp` file so that it prints us the Slater-parameters.
Here, we want to probe the interaction between two different sets of orbitals, therefore we specify in our cluster block that we want to evaluate two orbitals and then specify the two orbitals we are interested in. Here, we are interested in the 
3$`d`$ and the 2$`p`$ electrons, which we put in `energy set 2`. This means we write the cluster as
````
cluster
2 eV UJ
1 2 1 1 0  -1.0
1 1 2 1 0  -1.0
0  0  0.9
````
Here, the first line describes the 3$`d`$ electrons and the second the 2$`p`$ electrons.
We used solver 0 and a negative $U$ value, so that RSPt is not following our input, but instead calculates the Slaterparameters itself. While this probes the interaction between the $`p`$ and $`d`$ electrons we still need to tell RSPt to print it.
This is done with the `verbose` keyword `Umatrix`. One can use the `debug` keyword `Noscreening` to get the bare values of the Slater parameters, which one has to screen manually to ~80% (For pd and dd interactions).
If one does not do it, RSPt will apply a similar empirically motivated screening. 
````
verbose
Umatrix Dump

debug
Spin_avg_dc Noscreening Dump

````
The final `green.inp` file to calculate the Slater-parameters should look something like
````
inputoutput
F F

verbose
Umatrix Dump

debug
Spin_avg_dc Noscreening Dump

cluster
2 eV UJ
1 2 1 1 0  -1.0
1 1 2 1 0  -1.0
0  0  0.9

cluster
2 eV UJ
2 2 1 1 0  -1.0
2 1 2 1 0  -1.0
0  0  0.9
````

## Reading the output
After this has run for a single iteration, we will extract the Slater parameters from the `out` file.
In there search for the word `Slater` until you find a block like this:
````
 Slater parameters: cluster orbital set: i1,i2,i3,i4, F0 F2 F4 F6 or G1 G3 G5
    1   1   1   1 1.9469184 0.8937361 0.5540675 0.0000000
    1   1   2   2 0.3735431 0.2121888 0.0000000 0.0000000
    1   2   1   2 2.7285488 0.5065106 0.0000000 0.0000000
    1   2   2   1 0.3735431 0.2121888 0.0000000 0.0000000
    2   1   1   2 0.3735431 0.2121888 0.0000000 0.0000000
    2   1   2   1 2.7285488 0.5065106 0.0000000 0.0000000
    2   2   1   1 0.3735431 0.2121888 0.0000000 0.0000000
    2   2   2   2 8.4720677 3.9920178 0.0000000 0.0000000

````
Here, the first four columns are the indices in the $U$ matrix.
````
    1   1   1   1 
    1   1   2   2 
    1   2   1   2 
    1   2   2   1 
    2   1   1   2 
    2   1   2   1 
    2   2   1   1 
    2   2   2   2 

````
The 1 correspond to $`d`$ as the first line in our cluster in `green.inp` specified the 3$`d`$ electrons, while the 2 represents the 2$`p`$.
Therefore, the first line, which begins with ` 1 1 1 1` corresponds to $`U_{d,d,d,d}`$ and therefore $`F_{dd}`$. $`F_{pd}`$ is the direct Coulomb interaction between the 2$`p`$ and 3$`d`$ electrons, so $`U_{pdpd}`$, which is equivalent to $`U_{1212}`$.
Therefore, it can be found in the line starting with `1 2 1 2`.
The exchange interaction $`G_{pd}`$ can be found in the line beginning with ` 1 2 2 1`.  
Now, let us take a look at the actual values, which are written in columns 5 to 8. So they are given by:
```` 
1.9469184 0.8937361 0.5540675 0.0000000
0.3735431 0.2121888 0.0000000 0.0000000
2.7285488 0.5065106 0.0000000 0.0000000
0.3735431 0.2121888 0.0000000 0.0000000
0.3735431 0.2121888 0.0000000 0.0000000
2.7285488 0.5065106 0.0000000 0.0000000
0.3735431 0.2121888 0.0000000 0.0000000
8.4720677 3.9920178 0.0000000 0.0000000
````
Here, the first column indicates the $`F^0`$ and $`G^1`$ moments, the second row $`F^2`$ and $`G^3`$ moments and the third  $`F^4`$ and $`G^5`$. The elements of $`F^k`$ with an odd value of $k$ and the elements of $`G^k`$ with an odd value of $k$
are skipped as they have to be 0. We can also see that $`F_{dd}^6`$,  $`F_{pd}^4`$ and  $`G_{pd}^5`$ are zero. This is due to the fact that the $k$'s moment of $U$ for $`k > 2 \times \mathrm{min}(l,l')`$ is always 0.
Now, let us extract the Slater parameters, convert them to eV and screen them to 80%. This should give us:  
-$`F_{dd}^0`$= 21.18  
-$`F_{dd}^2`$= 9.72  
-$`F_{dd}^4`$= 6.03  
-$`F_{pd}^0`$= 29.69  
-$`F_{pd}^2`$= 5.51  
-$`G_{pd}^1`$= 4.06  
-$`G_{pd}^3`$= 2.31 
Here, we can clearly see that $`F^0_{dd}`$ and $`F^0_{pd}`$ are way to big to be realistic. This is due to the fact that the screening of this zeroth order moments is much stronger than for the higher order moments.
To estimate them we have to use other criteria. Here, we use the fact that $`F^0_{pd}`$ is just a different notation of the Hubbard $U$ parameter. So we can just use a scheme to determine that from $ab$ $initio$ or use the $U$ that best reproduces the band gap.
For $`F^0_{pd}`$ there are currently to our knowledge no calculation methods, but we are working on that.
For now, use the empirically motivated relation between $`F_{dd}^0`$ and $`F^0_{pd}`$, which is $`F^0_{dd}`$+1 eV $`\leq`$ $`F^0_{pd}`$ $`\leq`$$`1.3\times F^0_{dd}`$.
