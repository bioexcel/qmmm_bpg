==================================
Running QMMM simulations with CP2K
==================================


---------------------------------
Running a CP2K job
---------------------------------

A standard CP2K build creates multiple executables. It is advised to used the cp2k.psmp
executable as this allows the use of hybrid MPI+OpenMP calculations, which can have some
performance benefit over using MPI only on some machines.

CP2K should be run through a job submission script to run on the compute nodes.
The following command will launch CP2K (MPI only), specifing the input and output files, and the
number of mpi processes.

.. code-block:: none

  export OMP_NUM_THREADS=1
  joblauncher (-n procs) cp2k.pmsp -i inputfile.inp -o outputfile.out

The joblauncher will depend on the job launcher on your system, common examples are
mpiexec, srun and aprun. 

To run with multiple threads (MPI+OpenMP) the number of threads should be set to greater
than 1. Typical values where performance has been seen to be improved over pure MPI are 2, 4, 6, and 8
threads, although this will depend on many things such as your machine, your calcaultion type and
your system of interest. The number of threads to be chosen so that it evenly divides the number
of processes on a node, whilst ensuring that threads sharing memory are in the same NUMA region.
The number of processes will need to be set so that the threads



--------------------------
Performance considerations
--------------------------

A selection of CP2K QMMM benchmarks are available at: https://github.com/bioexcel/qmmm_benchmark_suite

The table below gives an overview of them.

Speed up plots for these on ARCHER are shown below in order to give an idea of a suitable number of cores
to use.

+-----------+---------------------+----------------+-------------+----------------+-----------------+
| Name      | Type                | QM atoms       | Total atoms | XC Functional  | Basis set       | 
+===========+=====================+================+=============+================+=================+
| MQAE      | solute-solvent      | 34             | 16,396      | BLYP	          | DZVP-MOLOPT-GTH | 
+-----------+---------------------+----------------+-------------+----------------+-----------------+
| ClC       | ion channel         | 19, 253        | 150,925     | BLYP	          | DZVP-MOLOPT-GTH |
+-----------+---------------------+----------------+-------------+----------------+-----------------+
| CBD_PHY   | phytochrome         | 68             | 167,922     | PBE            | DZVP-MOLOPT-GTH |
+-----------+---------------------+----------------+-------------+----------------+-----------------+
| GFP_QM    | fluorescent protein | 20, 32, 53, 77 | 28,264      | BLYP           | DZVP-GTH-BLYP   | 
+-----------+---------------------+----------------+-------------+----------------+-----------------+


ClC-19
------


.. image:: /_static/CIC-19-thread-improvements-su.png
    :width: 500px
    :align: center
    :height: 366px
    :alt: alternate text




