=====================
QMMM parameterisation
=====================

Choosing how to set up your QMMM region once you have selected your QM atoms

-----------------------
CP2K QMMM input format
-----------------------

The basic input for the QMMM section in CP2K is shown below. This contains all the parameters
describing the selection of QM atoms and how the QM region should be treated within the system.
This is designed to act as a rough guide for how to build your QMMM section, and contains some example
parameter settings with descriptions in the comments. It contains set up parameters as if to run
a QMMM calculation using the GEEP method. However other methods exist and these will require
slightly different parameters. These are explained under the ECOUPL option.


.. code-block ::

  &QMMM 
    &CELL
      ABC qm_x qm_y qm_z                 ! size of QM cell in x,y,z
      PERIODIC XYZ
    &END CELL
    ECOUPL GAUSS                         ! type of QM/MM elect. coupling
    USE_GEEP_LIB 9                       ! number of Gaussians in Gaussian expansion
    &PERIODIC                            ! apply periodic potential
      &MULTIPOLE ON                      ! turn on coupling of the QM multipole
      &END
    &END PERIODIC
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

    &LINK
       QM_KIND H 
       MM_INDEX  mm_atom_index           ! index of MM atom in broken bond
       QM_INDEX  qm_atom_index           ! index of QM atom in broken bond
       LINK_TYPE IMOMM                   ! IMOMM method
    &END LINK
 &END QMMM
    

------------------
QMMM Input options
------------------

ECOUPL
------

Specifies the type of the QM-MM electrostatic coupling. The following options are available:

* **COULOMB** -
* **GAUSS** - 
* **NONE** -
* **POINT_CHARGE** -
* **S-WAVE** -

USE_GEEP_LIB
------------

This allows use of the internal GEEP library to generate the gaussian expansion of the MM potential.
You can specify a number from 2 to 15, to set the number of gaussian funtions to be used in the expansion.

PERIODIC
---------

The periodic option can be  used to specify the periodicity of the QM cell.

PARALLEL_SCHEME
---------------

Chooses the parallel_scheme for the long range Potential. The choices are to parallelise
on the GRID or ATOM. ATOM is the default option, however this can require a lot of memory
as the grids are replicated, this is particularly the case when there 
Switching to the GRID scheme can reduce the memory requirements however when replicating
many atoms the performance may suffer. Instead you want to consider sticking with the ATOM
scheme, but using multiple threads per process or oversubscribing to increase the available 
memory.


CENTER
------

This sets when the QM system is automatically centered within the QM box. This can
prevent QM atoms from leaving the box. The options for this setting are EVERY_STEP, SETUP_ONLY
and NEVER.

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




Dealing with QM atoms moving outside of the cell
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

Walls can be added






-------------------------------
Dealing with the QM-MM boundary
-------------------------------



A localized orbital is introduced at the boundary between the regions

Once you have chosen the QM atoms you must deal with any bonds at the boundaries of the QM region,
between MM and QM atoms. This is to ensure that there are no dangling QM bonds.

Finding which bonds are cut
---------------------------

It is important that the bonds across the boundary are not expected to have large charge transfers,
as there is no treatment for charge transfer through the QMMM bounadary. Cutting a C-C bond for example
is usually a safe choice.

The bonds can be identied through visualisation, e.g. vmd or pdb viewer, or by observation
of the pdb file. To correctly treat a QM-MM bond in CP2K you need to know the atomic indexes
of the QM and MM atoms.

In CP2K there are x methods treat the QM boundary


Link atoms setup
----------------


Link atoms involve adding a atom (usually a hydrogen) to cap the QM bond in place of the MM atom.

.. code-block ::

    &LINK
       QM_KIND H 
       MM_INDEX  mm_atom_index           ! index of MM atom in broken bond
       QM_INDEX  qm_atom_index           ! index of QM atom in broken bond
       LINK_TYPE IMOMM                   ! IMOMM method
    &END LINK

