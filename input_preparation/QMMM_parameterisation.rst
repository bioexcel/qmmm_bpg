=====================
QMMM parameterisation
=====================

The chapter describes how set the QMMM settings in CP2K once you have decided on your QM atoms.
This includes how to get the QM atoms into the right format for CP2K, and how to parameterise
the QMMM cell. It will also deal with how to properly handle atomic bonds that cross the QMMM
boundary.

-----------------------
CP2K QMMM input format
-----------------------

The basic input for the QMMM section in CP2K is shown below. This contains all the parameters
describing the selection of QM atoms and how the QM region should be treated within the system.
This is designed to act as a rough guide for how to build your QMMM section, and contains some example
parameter settings with descriptions in the comments. It contains set up parameters as if to run
a QMMM calculation using the GEEP method. However other methods exist and these will require
slightly different parameters. These are explained under the ECOUPL option.


.. parsed-literal:: 

  &QMMM                            
    &CELL                                ! see :ref:`ref_qmmmcell`
      ABC qm_x qm_y qm_z                 ! size of QM cell in x,y,z
      :ref:`ref_periodic` XYZ
    &END CELL
    :ref:`ref_ecoupl` GAUSS                         ! type of QM/MM elect. coupling
    :ref:`ref_geep_lib` 9                       ! number of Gaussians in Gaussian expansion
    &PERIODIC                            ! apply periodic potential
      &MULTIPOLE ON                      ! turn on coupling of the QM multipole
      &END
    &END PERIODIC
    :ref:`ref_par_scheme` ATOM                  ! parallel scheme (atom is default)
    &QM_KIND N                           
      MM_INDEX list_of_atom_indexes      ! list of N QM atoms
    &END QM_KIND
    &QM_KIND H
      MM_INDEX list_of_atom_indexes      ! list of H QM atoms
    &END QM_KIND
    &QM_KIND C
      MM_INDEX list_of_atom_indexes      ! list of C QM atoms
    &END QM_KIND
    &QM_KIND O
      MM_INDEX list_of_atom_indexes      ! list of O QM atoms
    &END QM_KIND

    &LINK                                ! see :ref:`ref_link`
       QM_KIND H 
       MM_INDEX  mm_atom_index           ! index of MM atom in broken bond
       QM_INDEX  qm_atom_index           ! index of QM atom in broken bond
       LINK_TYPE IMOMM                   ! IMOMM method
    &END LINK
 &END QMMM
    
.. _ref_qmatoms:

-------------------
Setting up QM atoms
-------------------

The QM atoms need to be listed in terms of their MM index in the .pdb file 
(or coordinate file format of your choice). They should be grouped into their QM atomic
kinds (i.e. their element type) and given as a list of indexes, as shown below for N atoms.

.. code-block:: none

    &QM_KIND N                           
      MM_INDEX list_of_atom_indexes      ! list of N QM atoms
    &END QM_KIND

The QM kind should match a KIND specified in the SUBSYS section, which supplies the element
tpye and corresponding basis set and pseudopotential. To get the correct input format
for CP2K the get_qm_kinds.sh script is supplied which converts a pdb file containing the
QM atoms to this format. This pdb can be created by selecting the QM atoms from the whole system





--------------------------------
Important QMMM input parameters
--------------------------------

.. _ref_ecoupl:

ECOUPL
------

Specifies the type of the QM-MM electrostatic coupling. The following options are available:

* **COULOMB** - use a coulomb potential (not for GPW, GAPW).
* **GAUSS** - use a Gaussian expansion of the electrostatic potential (not for DFTB)
* **NONE** - use a classical point charge based coupling
* **POINT_CHARGE** - use a QM derived point charges
* **S-WAVE** - use a Gaussian expansion of the s-wave electrostatic potential

.. _ref_geep_lib:

USE_GEEP_LIB
------------

This allows use of the internal GEEP library to generate the gaussian expansion of the MM potential.
You can specify a number from 2 to 15, to set the number of gaussian funtions to be used in the expansion.

.. _ref_periodic:

PERIODIC
---------

The periodic option can be  used to specify the periodicity of the QM cell.

.. _ref_par_scheme:

PARALLEL_SCHEME
---------------

Chooses the parallel_scheme for the long range Potential. The choices are to parallelise
on the GRID or ATOM. ATOM is the default option, however this can require a lot of memory
as the grids are replicated, this is particularly the case when there 
Switching to the GRID scheme can reduce the memory requirements however when replicating
many atoms the performance may suffer. Instead you want to consider sticking with the ATOM
scheme, but using multiple threads per process or oversubscribing to increase the available 
memory.

.. _ref_center:

CENTER
------

This sets when the QM system is automatically centered within the QM box. 
The options for this setting are EVERY_STEP, SETUP_ONLY
and NEVER. The default is EVERY_STEP, which is suggested to prevent QM atoms from leaving the box.

.. _ref_qmmmcell:

--------------
QMMM Cell 
--------------

Selecting the size of the cell
------------------------------


The CELL section within the QMMM section contains setting for the QMMM cell which should contain the QM
atoms. This represents a boundary region where the MM atoms within it

QM atoms are by default centered within the cell so you do not have to worry about
its position within the cell for the whole system (this is controlled by the CENTER option).
However the dimensions of the CELL should be large enough to contain all the QM atoms.
A size roughly where the cell extends roughly 1.5-2A around the outermost QM atoms. 
If the CELL is much too small the QM energy will not be calculated properly and as a
consquence the SCF will not converge and/or the energies will be incorrect. 

To check the size of your CELL may want to consider running a series of energy calculations
at different cell sizes and check the convergence of the energy with the CELL size. Up to a certain size a larger cell
may be more accurate, however after this increasing the size further makes very little difference
to the energy, and will increase the run time.




Preventing QM atoms moving outside of the cell
------------------------------------------------

The QM atoms should stay within the QM box during a simulation. If they move outside
of the QM box the following warning message will be printed - "WARNING One or few QM atoms are within the SKIN 
of the quantum box". The calculation will usually continue in this case but the energies
and forces could be wrong.  This message will usually occur in the first few MD steps
of a simulation, and if you see this message it is a good idea to terminate the
calculation to check what might be wrong.

Some simple fixes for this might be to increase the size of the QM box and double 
check that the QM atoms are properly centered in the box using &QMMM&CELL&CENTERING.
However these options may not solve the issue if atoms are moving rapidly from within the box.
Fast movement of atoms in an MD simulation may be due to incorrect geometry. It can also happen if you 
have QM water atoms as these move around more readily
than protein atoms. In this case you have a few choices about how to prevent the
waters leaving the QM box.

.. **Constrain waters**

.. The water atoms in CP2K can be constrained in a similar way to those in classical
 MD simulation software. The 

**Add walls around QM box**

Walls can be added around the QM box to reflect any QM atoms which may try to leave the box.
This is of course, slightly unphysical so care should be taken to set this up in a way that preserves
the dynamics of the system. 






-------------------------------
Dealing with the QM-MM boundary
-------------------------------

Once you have chosen the QM atoms you must deal with any bonds at the boundaries of the QM region,
between MM and QM atoms. This is to ensure that there are no dangling QM bonds.

Finding which bonds are cut
---------------------------

It is important that the bonds across the boundary are not expected to have large charge transfers,
as there is no treatment for charge transfer through the QMMM bounadary. Cutting a C-C bond for example
is usually a safe choice.

The bonds can be identied through visualisation, e.g. vmd or other pdb viewer, or by observation
of the pdb file. To correctly treat a QM-MM bond in CP2K you need to know the atomic indexes
of the QM and MM atoms. The LINK section is then used to pass this information.


.. _ref_link:

QMMM Link parameterisation
--------------------------

The CP2K link treatment involves adding a atom (usually a hydrogen) to cap the QM bond in place of the MM atom.
This must be done for all dangling QM bonds or you will get the following error "

There are three different link treatments in CP2K which can be set using the LINK_TYPE option. These are as follows:

* GHO - Integrated Molecular Orbital Molecular Mechanics method
* IMOMM -  Generalized Hybrid Orbital method
* PSEUDO - Use a monovalent pseudo-potential

The element used to cap the bond can be changed by setting QM_KIND; the default option is hydrogen H.

An example LINK section is shown below:

.. code-block:: none

    &LINK
       QM_KIND H                         ! element capping
       QMMM_SCALE_FACTOR 1.0             ! scale factor of the MM charge
       MM_INDEX  mm_atom_index           ! index of MM atom in broken bond
       QM_INDEX  qm_atom_index           ! index of QM atom in broken bond
       LINK_TYPE IMOMM                   ! IMOMM method
    &END LINK


