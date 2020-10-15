==============================
 QM treatment
==============================

QM treatment of the QM region with QM/MM simualtion.

3 key considerations in running a QM simulation in CP2K, your choice of basis set
Puesdopotential and XC functional.  will guide on how to use various choices within 
cp2k but not how to make them

------------------------------
Mechanics of a DFT calculation
------------------------------

Quickstep - method

SCF - self consistent functional
calculated until the change in energy is less than a threshold
inner and outer scf - tolarences and max steps
basis sets + potentials
xc functional

---------------------------
Setting up basic QM input
---------------------------

A basic DFT input block for a calcualtion using the Gaussian Plane Wave (GPW) method. 
Examples for using a Semi-emperical method (SE) and the Tight Binding method (TDFT) are provided here:

.. code-block::

  &DFT                                 ! CP2K input block for DFT
    CHARGE 0                               ! charge of the QM region
    MULTIPLICITY 1                         ! multiplicityi of the QM region
    BASIS_SET_FILE_NAME  bs_filename       ! name of the basis set filename
    POTENTIAL_FILE_NAME  pot_filename      ! name of the potential filename

    &MGRID
      CUTOFF 300                           ! energy cutoff
      COMMENSURATE                         ! commensurate (always true for QMMM)
    &END MGRID
    
    &QS
      METHOD GPW                           ! QS method (GPW: DFT, DFTB, or SE method)
      EPS_DEFAULT 1.0E-12                  ! Energy tolarance for QS
    &END QS
    
    &SCF                               ! Parameters controlling the convergence of the scf.
      SCF_GUESS RESTART                    ! starting point for scf - restart reads restart wfn.
      EPS_SCF 1.0E-6                       ! tolarance for SCF convergence
      MAX_SCF 50                           ! number of inner SCF steps to attempt
      &OT  T
        MINIMIZER  DIIS                    ! DIIS minimisation (less reliable but faster than CG)
        PRECONDITIONER  FULL_ALL           ! good choice for preconditioner
      &END OT
      &OUTER_SCF
        EPS_SCF 1.0E-6                     ! tolarence for outer SCF convergence (should be > inner SCF)
        MAX_SCF 10                         ! number of outer SCF steps to attempt
      &END
    &END SCF
    &XC                                ! Parameters needed to compute the electronic exchange potential 
      &XC_FUNCTIONAL xc_choice             ! choice of XC functional (can simply change this for BLYP, PBE)
      &END XC_FUNCTIONAL
    &END XC

  &END DFT

Additionally for each element identifier in your topology you need to tell CP2K which basis 
sets and potentials to use. This is done in the SUBSYS section, under KIND. 

.. code-block::

  &SUBSYS
    &KIND H
      ELEMENT H
      BASIS_SET bs_identifier
      POTENTIAL pot_identifier
    &END KIND
  &END SUBSYS
 


------------
Basis sets
------------

The basis set can be changed by editing the bs_filename, and the bs_identifier 
under each element within the SUBSYS&KIND section. The bs_identifier should correspond
to one of the basis sets for the given element within the basis set file. Basis set
files are provided within the /data directory in CP2K (link).
If your install of CP2K  has been built correctly then
the files within this directory should be automatically included, so there is no
need to provide these in you working directory.


GTH - Geodecker tuetter hutter recommended.



---------------------
XC functionals
---------------------


---------------------
Puesdopotentials
---------------------


------------------------
Important QM parameters
------------------------

Cutoff
Rel_cutoff
NGRID
EPS_SCF
Commensurate

Converging cutoff/rel cutoff

-----------------
Troubleshooting
-----------------

My energy is positive
---------------------

SCF does not converge
---------------------

Some other CP2K error messages
------------------------------

