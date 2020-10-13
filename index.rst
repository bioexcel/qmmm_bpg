==============================
 Biomolecular QM/MM with CP2K
==============================

This guide provides practical advice on how to use CP2K to perform QM/MM simulation of biomolecular systems

------------
Introduction
------------

Warning: QM/MM is not a silver bullet, this guide covers how to use CP2K, not how to make modelling choices

Outline key steps, which are covered in sections below

---------------------
Preparing your system
---------------------

Preparing PDB
-------------
Solvating, setting charges



Forcefield preparation
----------------------
Anything needed to be done before you can simply specify it in CP2K input file




------------------------------------
Choosing QM/MM simulation parameters
------------------------------------

CP2K supports variety of QM treatment approaches/ theory levels (list)

MM forcefield
-------------


QM-MM region setup
------------------


Basis set
---------


QM treatment
------------


QM theory level 1
-----------------




QM theory level 2
-----------------





QM theory level 3
-----------------





------------
Running CP2K
------------


Parallel execution
------------------
MPI vs OpenMP, GPU 

Example execution commands (job scripts?)

Example compute times / scaling (QM/MM benchmark results) to guide HPC resource usage ("cookbook")

Outline CP2K performance regimes (SCF vs LS-SCF), GPU offloading, sparse vs dense linear algebra

Restart
-------

Understanding outputs
----------------------


Common errors
-------------




----------
References
----------
https://www.tandfonline.com/doi/full/10.1080/00268976.2017.1333644













