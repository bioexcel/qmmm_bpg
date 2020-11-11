==============
QMMM overview
==============


  
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


Before running a QMMM simulation in CP2K it is recommended to try and run a simple single energy
MM calculation in CP2K. This will verify that the input forcefield and coordinates
are set up correctly and compatible with CP2K before attempting to add more complicated
QMMM parameters. The MM calculation should be fast to run and if the calculation finishes without
error and the energy is sensible then it is a good indicator that the system has been
prepared correctly for CP2K.






------------------------------------------
Run a simple QMMM calculation in CP2K
------------------------------------------

Once an MM calcualtion has been run successfully the input can be used as a basis for a QMMM calculation.

The METHOD should be set to QMMM.
You will need to add the QMMM section for the parameterisation of the QM region and the DFT section
which will define all the necessary settings for the QM treatment. Additionally, information
about the atomic kind parameterisation will needed to be added for each kind in the SUBSYS section.

The structure of your input will look something like this:

.. code-block:: none


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

Information on setting up the parameters for the QMMM section can be found here:
Settings for this will depend highly on your choice of QM region.

Information on setting the QM treatment can be found here:
It is good practice to start with simple method for the XC functional and then check that the QM set up 
has been done correctly before increasing the complexity and deciding on most accurate or appropirate
method for your system.

You should first calculate just the ENERGY of the system and check that this is sensible and that the SCF
converges. This will ensure that there are not any errors in your DFT setup or QM atom selection.

Before running a production QMMM calculation the value of the CUTOFF should be converged
for the final choice of BASIS_SET, XC_FUNCTIONAL and any other parameters. How to do this
is documented here: https://www.cp2k.org/howto:converging_cutoff



----------------
Run MD with CP2K
-----------------

Once you  have setup a simple single energy QMMM calculation CP2K it fairly 
straightfoward to adjust the input file to run a production molecular dynamics simulation.


