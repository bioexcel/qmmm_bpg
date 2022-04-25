==============
Building CP2K
==============


This chapter will guide on how to install CP2K so that it can be used
with the GROMACS/CP2K interface. It will show you how to use the CP2K
toolchain to do this.


----------------
Downloading CP2K
----------------

CP2K can be downloaded from the `CP2K github repositry <https://github.com/cp2k/cp2k/releases/>`_


------------------
Set up environment
------------------

Usually on a HPC machine commonaly used software is available through
centrally installed modules. We suggest using these on your machine
if they are available rather than building your own version. Typically
CP2K recommends the use of the GCC compilers. You should also make 
sure that an MPI implemetation which is compatible with these is also 
loaded. If in doubt consult the documentation of the machine.

Apart from this you will also need

* BLAS, SCALAPACK, LAPACK
* Python3
* FFTW
* CMAKE

BLAS, SCALAPACK and LAPACK are usually available through Intel MKL library,
Cray-libsci, or OpenBlas/OpenSCALAPACK.

If building on a GPU you should also make sure that either the CUDA or
ROCM libraries are loaded.


----------------
CP2K toolchain
----------------


The simplest way to build CP2K if you are not familer with it is
with the CP2K toolchain. 

.. code-block:: none

  cd cp2k/tools/toolchain


The general way to invoke this is to use

.. code-block:: none

  ./install_cp2k_toolchain.sh -j

packages then be included by adding

.. code-block:: none
  --with-package=install

excluded by adding

.. code-block:: none
  --with-package=no

or directed to use the system version of the package by adding

.. code-block:: none
  --with-package=system

A lot of the default installed packages can be unnecssary. Therefore 
we suggest these be excluded from the build as these can make the
build process more complex.


For CPU and GPU builds the important options to set are the ``--math-mode`` and the ``--mpi-mode``

``--math-mode``, which sets the maths library, has the following choices:

* ``mkl`` - use Intel MKL
* ``acml`` - use the AMD core maths library
* ``cray`` - use cray-libsci on cray machines
* ``openblas`` - use openblas

``--mpi-mode``, which sets the mpi library, has the following choices:

* ``mpich``
* ``openmpi``
* ``intelmpi``
* ``no`` - disable mpi

Additionally for GPU builds you need to either use ``--enable-cuda`` for
Nvidia GPUs or ``--enable-hip`` for AMD GPUs, and also set the ``--gpu-ver``
to your GPU version (see below).



CPU options
-----------

.. code-block:: none

  ./install_cp2k_toolchain.sh --math-mode=<acml,cray,mkl,openblas> \
  --mpi-mode=<mpich,openmpi,intelmpi> --with-hdf5=no 
  --with-sirius=no --with-libvori=no --with-gsl=no --with-spfft=no --with-spglib=no 


GPU options
-----------

**CUDA**

.. code-block:: none

  ./install_cp2k_toolchain.sh --math-mode=<acml,cray,mkl,openblas> \
  --mpi-mode=<mpich,openmpi,intelmpi> --enable-cuda=yes --gpu-ver=<K20X, K40, K80, P100, V100> \
  --with-hdf5=no --with-sirius=no --with-libvori=no --with-gsl=no --with-spfft=no \
  --with-spglib=no 



**HIP**

.. code-block:: none

  ./install_cp2k_toolchain.sh --math-mode=<acml,cray,mkl,openblas> \
  --mpi-mode=<mpich,openmpi,intelmpi> --enable-hip=yes --gpu-ver=<Mi50, Mi10> \
  --with-hdf5=no --with-sirius=no --with-libvori=no --with-gsl=no --with-spfft=no \
  --with-spglib=no 


Enabling Plumed
---------------

If you would like to use Plumed for Metadynamics simulations in CP2K you can
add:

.. code-block:: none

  --enable-plumed=install

---------------
Compiling CP2K
---------------

After the toolchain has completed it will produce an environment setup file
in:

.. code-block:: none

  cp2k/tools/toolchain/install/setup

and a selection of arch files e.g. ``local.ssmp``, ``local.psmp``, ``local.popt``

The ``.psmp`` file which has MPI and threading enabled is the most useful of these.
You should first source the setup:

.. code-block:: none

  source install/setup

and then copy the arch files to cp2k/arch

.. code-block:: none

  cp install/arch/* ../../../../arch

For a CUDA or HIP build there will be a ``local_cuda.psmp`` or a ``local_hip.psmp``
arch file created in addition. You should use this if you 
wish to use the GPU offloading.

For the top level cp2k directory you can then compile with the arch file 
of your choice using:

.. code-block:: none

  make -j 12 ARCH=local VERSION=psmp

And then after this build libcp2k which is required for the GROMACS/CP2K
interface.

.. code-block:: none

  make -j 12 ARCH=local VERSION=psmp libcp2k


-----------------------
Building the interface
-----------------------


For information on building the GROMACS/CP2K interface please see the 
dedecated `GROMACS page <https://manual.gromacs.org/documentation/2022-beta1/install-guide/index.html#installing-with-cp2k>`_
