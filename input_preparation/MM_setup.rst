=================
MM setup
=================

An example CP2K input for a simple single energy MM calculation is shown below.
Note that the RUN_TYPE is set to ENERGY and the METHOD in the FORCE_EVAL section
is set to FIST for an MM run. This stands for Frontiers in Simulation Technology which is the 
method used in CP2K MM calculations.

.. code-block ::

  &GLOBAL
     PROJECT project_name
     RUN_TYPE ENERGY
     PRINT_LEVEL LOW
  &END GLOBAL
  &FORCE_EVAL
     METHOD FIST                       ! Do MM
     &MM
       &FORCEFIELD
       PARMTYPE AMBER                  ! use the Amber forcefield type
       DO_NONBONDED .TRUE.             ! short range non bonded interactions
       PARM_FILE_NAME ff_name          ! forcefield filename
       &SPLINE
          EMAX_SPLINE 1.0E14           ! max spline
          RCUT_NB [angstrom] 12        ! Cutoff radius for nonbonded interactions
       &END SPLINE
       &END FORCEFIELD
       &POISSON
          &EWALD
             EWALD_TYPE SPME           ! recommended ewald type
             GMAX 70                   ! number of grid points 1 per angstrom in each direction
          &END EWALD
       &END POISSON
     &END MM
     &SUBSYS
        &CELL
          ABC x y z                    ! size in x,y,z in Angstrom
          PERIODIC XYZ
          ALPHA_BETA_GAMMA 90 90 90    ! cubic cell (90 degrees between alpha, beta, gamma)
        &END CELL
        &TOPOLOGY                      
           CONN_FILE_FORMAT AMBER      ! use the amber format
           CONN_FILE_NAME ff_name      ! amber forcefield filename
           COORD_FILE_FORMAT PDB       ! coords in pdb formant (see also CRD, XYZ)
           COORD_FILE_NAME coord_name  ! coordinate filename
        &END TOPOLOGY
     &SUBSYS
  &END FORCE_EVAL



The MM and SUBSYS sections of FORCE_EVAL are required for this calculation. The MM section will contain 
all the parameters for MM such as the forcefield, and the poisson and spline information.
The subsys section contains the systems topology information
such as the atomic coordinates, the cell size and the connectivity.

--------------------
Forcefield types
--------------------




-----------------------
Important MM parameters
-----------------------


EMAX_SPLINE
-----------

RCUT_NB
-------

EWALD_TYPE
----------

GMAX
----

---------------
Troubleshooting
---------------

Warning about EMAX_SPLINE
-------------------------


KIND not found
---------------

You may get an error message from CP2K saying "". This is becasue CP2K only expects
proper element symbols in the coordinate and force field files. The work around for this is
to let CP2K know what element the symbol should correspond to. This is done by adding it as a KIND
in the SUBSYS section.

