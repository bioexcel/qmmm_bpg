=====================
QMMM parameterisation
=====================

Choosing how to set up your QMMM region once you have selected your QM atoms

-----------------------
CP2K QMMM input format
-----------------------

.. code-block ::

  &QMMM  			                     ! This defines the QS cell in the QMMM calc
    &CELL
      ABC qm_x qm_y qm_z                 ! size of QM box in x,y,z
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
    
    

--------------
QMMM Cell 
--------------

Selecting the size of the cell
------------------------------

QMMM&CELL

much too small SCF will not converge
usually add 1.5-2A around QM atoms
slighty too small energy different
converge energy with cell size, increase cell see energy change
much too big no real large effect on the energy but the run time is increased - wasteful



Dealing with QM atoms moving outside of the cell
------------------------------------------------

usually waters move readily around the box - having QM waters usually causes these kinds of errors

error message - WARNING in qmmm_util.F:: One or few QM atoms are within the SKIN 
of the quantum box. Check your run and you may possibly consider: the 
 activation of the QMMM WALLS around the QM box, switching ON the      
centering of the QM box or increase the size of the QM cell. CP2K    
 CONTINUE but results could be meaningless.                            


options
-------

constrain all waters

add walls around QM box - how to do this




-------------------------------
Dealing with the QM-MM boundary
-------------------------------

Finding which bonds to cut
---------------------------

usually c-c
look at pdb file
identify QM residue at end
find C-C bond
indexes


hydrogen capping dangling QM bond

Link atoms setup
----------------



------------------
QMMM Input options
------------------

ECOUPL
------

USE_GEEP_LIB
------------

MULTIPOLE
---------

CENTERING
---------
