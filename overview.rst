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
  
     &MM                                    ! MM section - MM forcefields,  etc.
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


Indentify ligend, region of interest

Residue numbers

growing/ shrinking

find broken bonds




----------------------------------
Run MM only in CP2K
----------------------------------

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
          ALPHA_BETA_GAMMA 90 90 90    ! cubic cell (90 degrees between alpha, beta, gamma
        &END CELL
        &TOPOLOGY                      
           CONN_FILE_FORMAT AMBER
           CONN_FILE_NAME ff_name      ! amber forcefield filename
           COORD_FILE_FORMAT PDB       ! coords in pdb formant (see also CRD, XYZ
           COORD_FILE_NAME coord_name  ! coordinate filename
        &END TOPOLOGY
     &SUBSYS
  &END FORCE_EVAL

Before running a QMMM simulation in CP2K it is recommended to try and run a simple single energy
MM calculation in CP2K. This will verify that the input forcefield and coordinates
are set up correctly and compatible with CP2K before attempting to add more complicated
QMMM parameters. The MM calculation should be fast to run and if the calculation finishes without
error and the energy is sensible then it is a good indicator that the system has been
prepared correctly for CP2K.

The FIST (Frontiers in Simulation Technology) method is used in CP2K MM calculations.
The MM and SUBSYS sections of FORCE_EVAL are required for this calculation. The MM section will contain 
all the parameters for the MM such as the forcefield, and the poisson and spline information.
The subsys section contains the systems topology information
such as the atomic coordinates, the cell size and the connectivity.





------------------------------------------
Run a simple QMMM calculation in CP2K
------------------------------------------

Once an MM calcualtion has been run successfully the input can be used as a basis for a QMMM calculation.

The METHOD should be set to QMMM.
You will need to add the QMMM section for the parameterisation of the QM region and the DFT section
which will define all the necessary settings for the QM treatment. Additionally, information
about the atomic kind parameterisation will needed to be added for each kind in the SUBSYS section.

Information on setting up the parameters for the QMMM section can be found here:
Settings for this will depend highly on your choice of QM region.

Information on setting the QM treatment can be found here:
It is good practice to start with simple method for the XC functional and then check that the QM set up 
has been done correctly before increasing the complexity and deciding on most accurate or appropirate
method for your system.

You should first calculate just the ENERGY of the system and check that this is sensible and that the SCF
converges.

Before running a production QMMM calculation the value of the CUTOFF should be converged
for the final choice of BASIS_SET, XC_FUNCTIONAL and any other parameters.



--------------
Run MD in CP2K
--------------

Once you  have setup a simple single energy QMMM calculation CP2K it fairly 
straightfoward to adjust the input file to run a molecular dynaimcs simulation.


