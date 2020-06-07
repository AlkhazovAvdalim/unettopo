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

## split\_data :

This Jupyter Notebook reads in the data from data\_modification,
processes it and saves it under different folders. The user has to
define an iteration x for the optimized structures. This is done as
follows:

1.  Load in the numpy arrays of the data instances one by one.

2.  Apply padding to the necessary matrixes (Matrices 0, 3-5).

3.  Split the data by Input (Matrices 0-5), Output (optimized structure
    from iteration x),Force Information. And save it into 3 folders.

This leads to 3 folders containing a numpy array for each data instance
with the according input, output and force matrices. If the iteration x
defined by the user exceeds the maximal iteration of the optimized
structure, the structure from the last iteration is used and the
loadcase is saved under under\_iteration.npy.

## unet\_kaggle\_wloop: 

This Jupyter Notebook is used in Kaggle Kernels to train the neural
network and create trained models. It loads in the split information
using a data generator, which operates by the individual paths of the
input and output data, which it then shuffles, loads and feeds into the
neural network incrementally. It allows to adjust the hyperparameters
and gives the trained model an ID, to identify it in a later comparison
process and the output is as follows:

1.  Image containing a plot of the loss through the epochs.
    
<img src="https://github.com/AlkhazovAvdalim/unettopo/blob/master/readme_images/ID_1_loss.png?raw=true" width="630" height="420">
2.  Image containing a plot of the binary accuracy through the epochs.
<img src="https://github.com/AlkhazovAvdalim/unettopo/blob/master/readme_images/ID_1_binary_accuracy.png?raw=true" width="630" height="420">    

3.  Image containing a plot of the dice coefficient through the epochs.
<img src="https://github.com/AlkhazovAvdalim/unettopo/blob/master/readme_images/ID_1_dice%20coefficent.png?raw=true" width="630" height="420">


4.  The history of the loss and other evaluations of the model.

5.  The trained model after the last epoch.

6.  The trained model at its lowest validation loss.

This notebook also allows training multiple models by using a loop for
the model IDs and hyperparameter adjustments.
