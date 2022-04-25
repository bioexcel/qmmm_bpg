===================================
Running QM/MM simulations with CP2K
===================================


---------------------------------
Running a CP2K job
---------------------------------

A standard CP2K build creates multiple executables. It is advised to use the cp2k.psmp
executable as this enables the hybrid MPI+OpenMP parallelisation scheme, which can have some
performance benefit over using only MPI parallelisation on some machines.

CP2K should be run through a job submission script to run on the compute nodes.
The following command will launch CP2K (MPI only), specifing the input and output files, and the
number of mpi processes.

.. code-block:: none

  export OMP_NUM_THREADS=1
  joblauncher (-n procs) cp2k.pmsp -i inputfile.inp -o outputfile.out

The joblauncher will depend on the job launcher on your system, common examples are
mpiexec, srun and aprun. 

To run with multiple threads (MPI+OpenMP) the number of threads should be set to a value greater
than 1. Typical values where performance may be improved over pure a MPI job are 2, 4, 6, and 8
threads, although this will depend on many things such as your machine architecture, the type of calculation and
your system size. As a general rule the number of threads per MPI process has to be chosen so that it evenly divides the number
of MPI processes on a node, whilst ensuring that threads sharing memory are in the same NUMA region.
The total number of MPI processes will need to be set so that the number of threads per process multiplied by the number of MPI
processes gives the total number of cores requested.



--------------------------
Performance considerations
--------------------------

A selection of CP2K QM/MM benchmarks are available at: https://github.com/bioexcel/qmmm_benchmark_suite

The table below gives an overview of them.


+-----------+---------------------+----------------+-------------+-------------------+-----------------+
| Name      | Type                | QM atoms       | Total atoms | XC Functional     | Basis set       | 
+===========+=====================+================+=============+===================+=================+
| MQAE      | solute-solvent      | 34             | 16,396      | BLYP, B3LYP       | DZVP-MOLOPT-GTH | 
+-----------+---------------------+----------------+-------------+-------------------+-----------------+
| ClC       | ion channel         | 19, 253        | 150,925     | BLYP, B3LYP, PBE0 | DZVP-MOLOPT-GTH |
+-----------+---------------------+----------------+-------------+-------------------+-----------------+
| CBD_PHY   | phytochrome         | 68             | 167,922     | PBE, PBE0         | DZVP-MOLOPT-GTH |
+-----------+---------------------+----------------+-------------+-------------------+-----------------+
| GFP_QM    | fluorescent protein | 20, 32, 53, 77 | 28,264      | BLYP              | DZVP-GTH-BLYP   | 
+-----------+---------------------+----------------+-------------+-------------------+-----------------+


Running on CPUs
---------------

The time per MD step (s) on ARCHER2 is reported in the Table below for the benchmark systems. ARCHER2 has 
128 cores per node, comprised of two 64-core AMD EYPC processors. More details are given on the [ARCHER2 website]
(https://www.archer2.ac.uk) The results below use MPI+OpenMP with 4 threads per MPI process which was found
to, in general, give the best performance.

+=======+==============+==============+=============+==============+================+=================+
| Cores | MQAE  (BLYP) | MQAE (B3LYP) | ClC (QM 19) | ClC (QM 253) | CBD_PHY (PBE)  | CBD_PHY (PBE0)  |
+=======+==============+==============+=============+==============+================+=================+
| 32    | 12.86        | 20.94        | 46.83       | 168.16       | 112.83         | 266.25          |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+
| 64    | 7.38         | 12.38        | 25.82       | 111.86       | 66.95          | 127.51          |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+
| 128   | 4.91         | 7.85         | 15.07       | 75.03        | 38.13          | 69.48           |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+
| 256   | 3.55         | 5.13         |             | 57.79        | 24.89          | 40.21           |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+
| 512   |              |              |             |              |                | 24.93           |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+

Running on GPUs
---------------

The time per MD step (s) on Cirrus GPU nodes is reported in the Table below for the benchmark systems.
The Cirrus GPU nodes contain 4 GPUs per node and 20 CPU cores. The GPUs are Nvidia Volta V100's
Here we assign one MPI process per GPU and 10 OpenMP threads per process to make use of the CPU cores. 
 More details are given in the [Cirrus documentation](https://cirrus.readthedocs.io/en/main/user-guide/gpu.html) 

Using the GPU enabled [COSMA library](https://github.com/eth-cscs/COSMA) was found to not significantly 
improve the performance.

+=======+==============+==============+=============+==============+================+=================+=================+
| Cores | GPUs         | MQAE  (BLYP) | MQAE (B3LYP) | ClC (QM 19) | ClC (QM 253)   | CBD_PHY (PBE)   | CBD_PHY (PBE0)  |
+=======+==============+==============+=============+==============+================+=================+=================+
| 40    | 4            | 15.43        | 25.55       | 55.73        |                | 136.09          |                 |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+-----------------+
| 80    | 8            | 9.47         | 14.32       | 32.80        |                | 89.75           | 155.98          |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+-----------------+
| 160   | 16           | 6.44         | 8.91        | 20.31        | 63.08          | 38.66           | 70.43           |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+-----------------+
| 320   | 32           |              | 6.36        |              | 46.62          | 24.79           | 41.19           |
+-------+--------------+--------------+-------------+--------------+----------------+-----------------+-----------------+


ClC-19
------


.. image:: /_static/CIC-19-thread-improvements-su.png
    :width: 500px
    :align: center
    :height: 366px
    :alt: alternate text




