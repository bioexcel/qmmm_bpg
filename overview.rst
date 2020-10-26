==============
QMMM overview
==============


----------------------
Overview of CP2K input
----------------------

.. code-block ::


  &GLOBAL                                   ! global section (required)
     PROJECT project_name                      ! name of project for output files
     RUN_TYPE ENERGY                           ! run type (important)
     PRINT_LEVEL LOW                           ! controls amount of output produced
  &END GLOBAL

  &FORCE_EVAL                               ! parameters for force evaluation
     METHOD QMMM                               ! method employed e.g. QMMM, QS, FIST
     
     &DFT                                   ! DFT section - all QM 
       .... contents of DFT section
     &END DFT
  
     &QMMM                                  ! QMMM section - set up for QM region
       .... contents of QMMM section
     &END QMMM
  
     &MM                                    ! MM section - MM forcefields, connectivity etc.
       .... contents of MM section
     &END MM
     
     &SUBSYS                                ! subsystem - coordinates, atom kinds etc.
       .... contents of SUBSYS section
     &SUBSYS
     
  &END FORCE_EVAL
   
  &MOTION                                   ! control of atom movement e.g. geometry optimisations, MD
    .... contents of MOTION section
  &END MOTION
  
---------------
Prepare system
---------------




----------------------------------------------
Minimisation, Equilibration and Thermalisation
----------------------------------------------


---------------
Select QM atoms
---------------







----------------------------------
Test run of MM only in CP2K
----------------------------------

.. code-block ::

  &GLOBAL
     PROJECT project_name
     RUN_TYPE ENERGY
     PRINT_LEVEL LOW
  &END GLOBAL
  &FORCE_EVAL
     METHOD FIST                       ! MM method
     &MM
       &FORCEFIELD
       PARMTYPE AMBER                  ! use the amber forcefield 
       DO_NONBONDED .TRUE.             ! short range non bonded interactions
       PARM_FILE_NAME ff_name          ! forcefield filename
       &SPLINE
          EMAX_SPLINE 1.0E14           ! max spline
          RCUT_NB [angstrom] 12        ! Cutoff radius for nonbonded interactions
       &END SPLINE
       &END FORCEFIELD
       &POISSON
          &EWALD
             EWALD_TYPE SPME           ! recommended in cp2k manual
             GMAX 70                   ! number of grid points 1 per angstrom in each direction
          &END EWALD
       &END POISSON
     &END MM
     &SUBSYS
        &CELL
          ABC x y z                    ! size in x,y,z
          PERIODIC XYZ
          ALPHA_BETA_GAMMA 90 90 90    ! cubic cell
        &END CELL
        &TOPOLOGY                      
           CONN_FILE_FORMAT AMBER
           CONN_FILE_NAME ff_name      ! amber forcefield filename
           COORD_FILE_FORMAT PDB       ! coords in pdb formant (see also CRD, XYZ
           COORD_FILE_NAME coord_name  ! coordinate filename
        &END TOPOLOGY
     &SUBSYS
  &END FORCE_EVAL

FIST

input pdb and topology
cell size






------------------------------------------
Run a simple QMMM calculation in CP2K
------------------------------------------


Add in sections QM and QMMM - see instructions

Energy only calculation

Check this works - converges, energy is negative


--------------
Run MD in CP2K
--------------


