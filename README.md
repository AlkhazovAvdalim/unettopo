# unettopo

## data\_modification :

A big problem for the practical application of this project was the data
size. The data used for this project took around 2 TB in OptiStructform.
Therefore, a program was written to extract the necessary information
out of the OptiStructform. data\_modification.ipynb reads in the
OptiStructdata containing the required mechanical information,
transforms this data and saves it as a numpy array, ready to be
processed further. This array is structured as followed (with regard to
the array indexing) :

1.  Matrix containing the Volume Fraction

2.  Matrix containing the X-Displacements (0th iteration)

3.  Matrix containing the Y-Displacements (0th iteration)

4.  Matrix containing the X-Strains (0th iteration)

5.  Matrix containing the Y-Strains (0th iteration)

6.  Matrix containing the XY-Strains (0th iteration)

7.  This entry contains all of the optimized structures from the 0th to
    the last iteration.

8.  Matrix containing the X-Forces

9.  Matrix containing the Y-Forces

10. Matrix containing the Forces in this type: "node\_i,
    x\_force\_value\_i, y\_force\_value\_i"

The main purpose of this program is to shrink the data, while conserving
all the necessary information. Applying this program on the test data
decreased the data from approximately 2 TB to 30 GB. After the data is
shrunk it can be processed further. It creates a checkpoint
checkpoint.npy, so if the kernel is interrupted and restarted the data
modification proceeds at a similiar spot at which it was interrupted.
The faulty.npy contains loadcases in which it was not possible to
convert the data, for example if the OptiStructdata is missing
information.

