==============
QMMM overview
==============

The following overview will outline the main steps in setting up and running a QMMM 
simulation in CP2K, starting from preparing your system from a raw pdb file, and leading 
towards running a production QMMM run, for example a molecular dynamics simulation. Each
section will contain relavent links to the specfic parts of the guide.

This overview is designed to be read from the point of view of a beginner, a more experianced
user may benefit from using the contents page to directly navigate to areas of interest.

  
---------------
Prepare system
---------------

The first step is to prepare your system - i.e. to generate the desired starting 
system and forcefield files. The starting point for this is usually a pdb file 
which may have been downloaded from the PDB data bank. Usually these structures are not
ready to use and will require fixing to obtain the desired protonation state of the protein.
Details of how to this are described in the system prepartion guide here: 
:doc:`../system_preparation/system_preparation`


You will also have to generate a forcefield for you system. There are different tools
available to do this depending on the type of forcefield. For CP2K the Amber, Charmm and Gromos
forcefields can be used. The relevant tools and their tutorials can be found below:

- Ambertools - https://ambermd.org/tutorials/ForceField.php
- Charmm - https://www.charmm.org/charmm/documentation/tutorials/
- Gromos -


----------------------------------------------
Minimisation, Equilibration and Thermalisation
----------------------------------------------

After you have built your topology and coordinate files you must minimise your system. 
This will find the energy minima for your system and fix any possible bad contacts in your initial structure.
Once the system is minimised, it has to be subsequently heated (from 0K to your target conditions i.e. 300K ) and equilibrated. 
This gradual and slow heating process prevents instabilities arising due to the
sudden increase in the kinetic energy.


---------------
Select QM atoms
---------------

After the system has been prepared you can consider which atoms will be included 
in the QM region. This will usually be the atoms in your ligand, however it is 
important to consider whether enough atoms are included to accurately represent
the chemistry of the system, while at the same time not having too many QM atoms 
(which will require more compute resources).

Details on how to select QM atoms from a pdb file and get them in the correct format
for using in a CP2K input are given here: :doc:`../system_preparation/selecting_qm_atoms`




----------------------------------
Run MM only in CP2K
----------------------------------


Before running a QMMM simulation in CP2K it is recommended to try and run a simple single energy
MM calculation in CP2K. This will verify that the input forcefield and coordinates
are set up correctly and compatible with CP2K before attempting to add more complicated
QMMM parameters. The MM calculation should be fast to run and if the calculation finishes without
error and the energy is sensible then it is a good indicator that the system has been
prepared correctly for CP2K. 

Details of how to this are given here: :doc:`../input_preparation/MM_setup`

If this is your first time using CP2K then it is recommended to read the Running CP2K section of guide (:doc:`../running_cp2k/running_cp2k`),
as well as the CP2K Output Guide (:doc:`../running_cp2k/cp2k_output`). If you encouter any errors while runnning
you can debug these in the Troubleshooting guide (:doc:`../running_cp2k/cp2k_errors`).






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

Information on setting up the parameters for the QMMM section can be found here: :doc:`../input_preparation/QMMM_parameterisation`

Settings for this will depend highly on your choice of QM region.

Information on setting the QM treatment can be found here: :doc:`../input_preparation/QM_treatment`

It is good practice to start with simple method for the XC functional and then check that the QM set up 
has been done correctly before increasing the complexity and deciding on most accurate or appropirate
method for your system.

You should first calculate just the ENERGY of the system and check that this is sensible and that the SCF
converges. This will ensure that there are not any errors in your DFT setup or QM atom selection.

Before running a production QMMM calculation the value of the CUTOFF should be converged
for the final choice of BASIS_SET, XC_FUNCTIONAL and any other parameters. How to do this
is documented here: https://www.cp2k.org/howto:converging_cutoff

.. -----------------------------------------
.. Running a Geometry Optimisation with CP2K
.. -----------------------------------------


.. https://www.cp2k.org/howto:geometry_optimisation

-----------------
Run MD with CP2K
-----------------

Once you  have setup a simple single energy QMMM calculation CP2K it is fairly 
straightfoward to adjust the input file to run a production molecular dynamics simulation.

The first change is to set the ``RUN_TYPE`` to MD. You will also need to add an MD section 
in the MOTION section which will list the parameters to do with the dynamics of the 
simulation. For a simple NVE MD ensemble this would look like this:

.. code-block:: none

 &MOTION
    &MD
       ENSEMBLE NVE                            ! Ensemble type
       STEPS 5                                 ! Number of MD steps
       TEMPERATURE 300                         ! Target temperature in Kelvin
       TIMESTEP 1                              ! Timestep in femtoseconds
    &END MD
 &END MOTION 
 
Usually a timestep of 1 femtoseconds or less is recommended in order to ensure
energy conservation in the system.

More information about MD simulations in CP2K is 
given here: https://www.cp2k.org/howto:md


Ensembles
------------


CP2K offers a range of MD ensembles which are listed here: https://manual.cp2k.org/trunk/CP2K_INPUT/MOTION/MD.html#ENSEMBLE

Common ones are the NVT and NPT_I (iosbaric) ensemble. For an NVT ensemble 
you will need to add information about the thermostat in the &THERMOSTAT section
within the MD section, and the NPT_I ensemble will need both and THERMOSTAT and 
BARASTAT section as shown below.

.. code-block:: none

 &MOTION
 ..
  &MD
     ENSEMBLE NPT_I
     TIMESTEP  0.5
     STEPS  1000 
     TEMPERATURE 298
     &BAROSTAT
       TIMECON [fs] 100        ! timeconstant for barostat
       PRESSURE [bar] 1.0      ! target pressure
     &END BAROSTAT
     &THERMOSTAT
       TYPE CSVR               ! type of thermostat -  options include NOSE, CSVR (rescaling), GLE, AD_LANGEVIN
       &CSVR
         TIMECON [fs] 10.      ! time constant for thermostat
       &END CSVR
     &END THERMOSTAT
  &END MD
  ..
 &END MOTION


.. -----------------------------------
.. Running a NEB calculation with CP2K
.. -----------------------------------
