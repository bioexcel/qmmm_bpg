=============================
Understanding the CP2K output
=============================

-------------------------
General Output 
-------------------------

Standard output
---------------

The beginning of the standard output contains information about the setup of the run. 
This will  print the important input parameters and details of how CP2K was built and run
e.g. the number of processes and threads that were specifed when running. It can be
used for future reference.

After this part, the output will then print details of the individual setup for the important
sections in the input file. These will appear under clear headings e.g. FIST for the MM section, 
Quickstep for the DFT section, and QMMM. Input parameters are reported, along with some more information
such as for example the number of basis functions.

The report of the calculation then follows in the output file. It will start with the
results from the self-consistent Kohn-Sham ground state calculation (SCF optimisation).
This will look similar to the snippet below.


.. code-block:: none

 SCF WAVEFUNCTION OPTIMIZATION

  ----------------------------------- OT ---------------------------------------
  Minimizer      : DIIS                : direct inversion
                                         in the iterative subspace
                                         using   7 DIIS vectors
                                         safer DIIS on
  Preconditioner : FULL_ALL            : diagonalization, state selective
  Precond_solver : DEFAULT
  stepsize       :    0.15000000                  energy_gap     :    0.08000000
  eps_taylor     :   0.10000E-15                  max_taylor     :             4
  ----------------------------------- OT ---------------------------------------

  Step     Update method      Time    Convergence         Total energy    Change
  ------------------------------------------------------------------------------
  Decoupling Energy:                                              -0.0015860332
  Recoupling Energy:                                               0.0000002197
     1 OT DIIS     0.15E+00 1261.9     0.00000101      -412.4425164003 -4.12E+02
  Decoupling Energy:                                              -0.0015862541
  Recoupling Energy:                                               0.0000002197
     2 OT DIIS     0.15E+00    1.0     0.00000216      -412.4425163985  1.73E-09
  Decoupling Energy:                                              -0.0015860586
  Recoupling Energy:                                               0.0000002197
     3 OT DIIS     0.15E+00    1.0     0.00000051      -412.4425164055 -6.98E-09


The OT block prints information about the orbital transform solver. This again,
reports the options set in the input file.
 
The SCF steps are then printed as they are calculated, listing the Step number, Method,
run Time, Convergence, Total Energy and the Change in energy from the last step. 
The first step will usually take longer as there are set up overheads.

In an ideal calculation the Total Energy should decrease with each step
as the energy is minimised (this means the Change should be negeative). 
The Change in energy should also decrease as it converges to the final value.
The Convergence gives an idea of the accuracy of the SCF energy. When this value 
becomes less than the ``EPS_SCF`` value specified in the input the SCF calculation will
exit and a message stating that convergence has been acheived is printed.

.. code-block:: none

 *** SCF run converged in     3 steps ***

Otherwise, if the SCF does not converge, it continues until the number of steps equals the MAX_SCF in the
input file. The inner SCF loop will finish and then move to the next outer SCF step, 
recalculating the preconditioner. Finally, if the number of outer SCF steps equals to
the corresponding ``MAX_SCF`` value in the input file, then the SCF calculation has not converged,
a warning will be printed to the standard output and the SCF will be abandoned.
Information on how to handle this is given in the troubleshooting chapter.

Upon successful convergence, electronic densities and energies (and forces if specified) information is printed.


.. code-block:: none

  Electronic density on regular grids:       -213.9999999972        0.0000000028
  Core density on regular grids:              213.9999999999       -0.0000000001
  Total charge density on r-space grids:        0.0000000026
  Total charge density g-space grids:           0.0000000026

  Overlap energy of the core charge distribution:               0.00000923326382
  Self energy of the core charge distribution:               -943.25295781363013
  Core Hamiltonian energy:                                    284.50150313570356
  Hartree energy:                                             355.46908698187769
  Exchange-correlation energy:                                -85.84000086143999
  Hartree-Fock Exchange energy:                               -18.79624573644062
  QM/MM Electrostatic energy:                                  -4.52391134484379

  Total energy:                                              -412.44251640550942

  outer SCF iter =    1 RMS gradient =   0.51E-06 energy =       -412.4425164055
  outer SCF loop converged in   1 iterations or    3 steps


  ENERGY| Total FORCE_EVAL ( QMMM ) energy (a.u.):          -2134.963290942846925

It is a good idea to check that the electronic density corresponds to the number of 
electrons, and that the charge is as expected.

The Total energy given is the energy from only the QM part i.e from the SCF calculation.
A breakdown of its components is printed above it. The total number of outer SCF loops
and inner SCF steps that were done is also shown.

The ``ENERGY (QMMM)``` is the QM/MM energy including all its components; the QM energy, MM energy
and QM-MM interation energy. This is the energy you are usually interested in.


Wavefuntions - NAME-RESTART.wfn
----------------------------------

Wavefunction files are binary files that contain the wavefunctions obtained from the last SCF step.
They are named with the project_name preceeding ``'-RESTART.wfn'``.
One is written every SCF step, and if a wavefuntion file of the same name
already exists the older version is moved to ``NAME-RESTART.wfn.bak-1``, rather than overwritten.
This is done for up to three files and so you may see the following files, where
the third backup (bak-3) is the oldest.

.. code-block:: none

 NAME-RESTART.wfn
 NAME-RESTART.wfn.bak-1
 NAME-RESTART.wfn.bak-2
 NAME-RESTART.wfn.bak-3

Wavefunction restarts are  used when restarting a calculation in order to act
as a guide for the first SCF step to speed up the calculation.
In this case the project name should be consistent
and the ``SCF_GUESS`` should be set to 'restart'. Care should be taken that
the wavefunction is a suitable guess for the SCF calculation otherwise it may not
converge. 

.. -----------------------------------
.. Output from a Geometry optimatision
.. -----------------------------------

.. Standard output
.. ---------------


.. Geometry trajectory 
.. --------------------

.. This is usually printed in an xyz file or similiar. It will show how the atomic coordinates
.. are changing throughout the optimisation. The coordinates of subsequent steps are printed
.. one after the other in the file. This can be viewed as a .xyz movie in order to view the progression.
.. Once the optimation has completed the final image in the file represents the optimised
.. configuration.

.. .restart file
.. --------------



---------------------
Output from an MD run
---------------------


.ener file
----------

The .ener file shows the important information about the system as the simulation 
progresses. For each step the Temperature, Kinetic energy, Potential energy, CPU time
and the Conserved Quantity are printed. The Conserved Quantity can represent different
quantities according to the statistical ensemble we are sampling on. For example, if we
are running an NVE simulation the Conserved Quantity corresponds to the total energy of
the system of interest, while if we run an NVT simulation then the Conserved Quantity
is the total energy of a system that includes the system of interest and also the
thermostat degrees of freedom.


MD trajectory
-------------

This will look similar to the geometry trajectory from a geometry optimation and 
will appear as a .xyz file as default. The coordinates at each MD step are printed in order
in this file. You can control how often this file is updated with new coordinates by using the
following settings within the motion section:

.. code-block:: none

 &MOTION
    &PRINT
       &TRAJECTORY
         &EACH
            MD 10                       ! print every 10 steps
         &END EACH
       &END TRAJECTORY
    &END PRINT
 &END MOTION

.restart file
-------------

The .restart file is written at the end of an MD step and it contains the information
to restart the MD simulation from that step. This file is human readable and reports information about the system 
setup along with the current positions and velocities of the atoms in the system.

To restart an MD simulation the ``EXT_RESTART`` section has to be added to the input file to instruct the code
to use the .restart file as the restart point:

.. code-block:: none

  &EXT_RESTART
     EXTERNAL_FILE_NAME  project-1.restart
  &END EXT_RESTART

You can also use the ``.restart`` file as if the input file for restarting the simulation,
supplying it directly to the CP2K executable in the job submission script:

``cp2k.psmp –i project-1.restart``
