# UnetTopo

## Abstract

In this project, a method from machine learning is proposed to
accelerate the pixel-based topology optimization. The goal is to develop
a software, which shows a proof of concept for using a neural network to
predict mechanical structures based on mechanical information passed
into the neural network as input. Another goal of this project was to
compile a compendium of deep learning in general for mechanical
engineers.

The software developed for this project modifies raw data from
Optistruct(TM) form, the conventional tool used in the chair for optimization
of mechanical structures, trains a neural network and is used after the
training of the neural network to predict new data instances, which are
then brought back into the conventional Optistruct(TM) form.

The input of the neural network consists of the following six matrices:
Volume Fraction, X-Displacement, Y-Displacement, X-Strains, Y-Strains,
XY-Strains. The output of the neural network is a matrix containing
predicted densities for each structure element. 19,600 load cases with
the dimension 64 x 48 have been used for the training and
validation. The architecture of the used network is heavily based on the
architecture proposed in Yiquan Zhang et al. “A deep Convolutional
Neural Network for topology optimization with strong generalization
ability”.

The structure optimization using the neural network is around 31 times
faster with regard to the whole inference process and around 838 times
faster with regard to only the prediction process. The quality of these
predictions was worse compared to the conventional method with an
observed factor of 1.32-1.45 in the 75% quartile with an extreme case
reaching a factor of over 200. (n=185)

Therefore it is not suggested to use the neural network in its current
state as a reliable stand-alone optimization tool, but use the predicted
structures as an initial design for the conventional method of
mechanical structure optimization and incrementally retraining the
neural network with each mechanical optimization.

## Handbook

### data\_modification :

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

### split\_data :

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

### unet\_kaggle\_wloop: 

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

### compare\_model\_losses: 

This Jupyter Notebook is used to compare the losses of the different
models. It reads in the corresponding history files and creates a plot
with the losses of the different models plotted in the same graph.
<img src="https://github.com/AlkhazovAvdalim/unettopo/blob/master/readme_images/ID_116_vs_ID_117_vs_ID_118_loss.png?raw=true" width="630" height="420">

<img src="https://github.com/AlkhazovAvdalim/unettopo/blob/master/readme_images/ID_116_vs_ID_117_vs_ID_118_val_loss.png?raw=true" width="630" height="420">

### compare\_structure\_predictions: 

This Jupyter notebook will load in different models and predict several
example loadcases with them. After the predictions are made, a figure is
created, which allows to see the difference in the structure’s model by
model and compared to the ground truth exactly as in the following
figure:

<img src="https://github.com/AlkhazovAvdalim/unettopo/blob/master/readme_images/Loadcase%20LoadCase_7852.npy101_vs_102_vs_103.png?raw=true" width="456" height="1200">

### inference: 

This is the final Jupyter notebook, which is used once a successful
model is chosen. It is used to modify, split, and predict new
OptiStructdata. For that it uses two of the previously mentioned
notebooks in adjusted versions:

  - data\_modification\_inf: for this notebook, only the 6 first
    matrices are needed, since the output is irrelevant.

  - split\_data\_inf: this notebook has been repurposed to apply padding
    to the necessary matrixes.

There are two additional resources this notebook uses:

  - create\_output\_txt: this notebook has a function to read in the
    split data and create predictions using the neural network. After
    predicting the structures it creates a .txt file containing the
    prediction in the following structure: "ELEMENT ID" (devined by
    OptiStruct) and "PREDICTED DENSITY OF ELEMENT" for all the elements
    and data instances. After this .txt file is made, the "change.fem"
    containing the volume fraction restriction and forces is copyied for
    each individual instance and the "inputDeck.fem", which contains the
    arrangement of the element IDs. These three files are necessary to
    run "Avdalim\_OptiStruct.exe".

  - Avdalim\_OptiStruct.exe: this .exe has been written by Mr. Philipp
    Clemens of the Chair of Optimization of Mechanical Structures and
    creates two new InputDeck.fem files containing the structure
    predicted by the neural network. One of these files has a penalty
    factor built in, which penalizes structures with many uncertain
    densities and one without the penalty factor. This allows not only
    to evaluate the performance of the neural network, but also makes it
    possible to use the predicted structure as initial design for the
    conventional mechanical optimization.

To use this notebook an inference folder has to be created with the
following subfolders:

  - data: this folder contains another folder called "raw", to which the
    raw OptiStructloadcase folders have to be copied to.

  - utils: contains the model and the Avdalim\_OptiStruct.exe.

  - temp: contains the modified and split data.

  - out: is created in the output process. It contains folders for all
    the loadcases with two subfolders: "WithoutP" and "WithP". In these
    subfolders are the new InputDeck.fem files to pass into OptiStruct.
