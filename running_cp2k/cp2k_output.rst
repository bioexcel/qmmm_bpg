=============================
Understanding the CP2K output
=============================

-------------------------
General Output 
-------------------------

Standard output
---------------

The start of the standard output contains information about the set up of the run. 
This will  print the important input paramters and details of how CP2K was built and run
e.g. the number of processes and threads that were specifed when running. It can be
used for future reference.

After this it will then print details of the individual set up for the important
sections in the input file. These will appear under clear headings e.g. FIST for MM, 
Quickstep for DFT, and QMMM. Input parameters are printed, along with some more information
such as for example the number of basis functions.

After this the output from the calculation is usually printed. This will start with the
output from the self-consistent Kohn-Sham ground state calculation (SCF optimisation).
This will look similar to the below.


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
reiterates the options set in the input file.
 
The SCF steps are then printed as they are calculated, listing the Step number, Method,
run Time, Convergence, Total Energy and the Change (in energy from the last step). 
The first step will usually take longer as there are set up overheads.

In an ideal calculation the Total Energy should decrease with each step
as the energy is minimised (this means the Change should be negeative). 
The Change in energy should also decrease as it converges to the final value.
The Convergence gives an idea of the accuracy of the SCF energy. When this value 
becomes less than the EPS_SCF value specified in the input the SCF calculation will
exit and a message stating that convergence has been acheived is printed.

.. code-block:: none

 *** SCF run converged in     3 steps ***

Alternatively if the SCF does not converge SCF steps will continue until the MAX_SCF is 
reached. The inner SCF loop will finish and then move on to the next outer SCF step, 
recalcailting the preconditioner. If then the outer scf MAX_SCF steps are exceeded then
and the SCF calculation has not converged a warning will be printed and the SCF will be abandoned.
Information on how to handle this is given in the troubleshooting chapter.

Upon successful convergence energy (and force if specified) information is printed.


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

The ENERGY (QMMM) is the QMMM energy including all its components; the QM energy, MM energy
and QMMM interation energy. This is the energy you are usually interested in.


Wavefuntions - NAME-RESTART.wfn
----------------------------------

Wavefunction files are binary files that contain the wavefunctions obtained from the last SCF step.
They are named with the project_name preceeding '-RESTART.wfn'.
One is written every SCF step, and if a wavefuntion file of the same name
already exists the older version is moved to NAME-RESTART.wfn.bak-1, rather than overwritten.
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
and the SCF_GUESS should be set to 'restart'. Care should be taken that
the wavefunction is a suitable guess for the SCF calculation otherwise it may not
converge. 

-----------------------------------
Output from a Geometry optimatision
-----------------------------------

---------------------
Output from an MD run
---------------------


.ener

.restart
