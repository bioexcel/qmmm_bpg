==============================
 QM treatment
==============================

The section of the guide will focus on setting up the QM parameters in a QM/MM simualtion.


It will guide on how to use various options within CP2K, and give some general advice but
it will not give any information about what choices are best for your system.

--------------------------------------
Mechanics of a DFT calculation in CP2K
--------------------------------------

CP2K uses the Quickstep (QS) method when performing a QM calculation.
Quickstep is the particular electronic stucture method within CP2K which handles
the calculation of forces and energies on the QM atoms. This is done using 
self-consistent Kohn-Sham Density Functional Theory (DFT). Within Quickstep
there is the option to use Semi-empirical (SE), Tight-Binding DFT (DFTB),
and Gaussian Planewaves (GPW) methods, among many others. The GPW method is a 
key part of CP2K, this involves using a mixed basis of atom centred Gaussian
orbitals and plane waves (regular grids) to improve on the performance compared
to other QM methods. You can find information about GPW here:

Basis functions descibted in the Gaussian type orbitals for each element are supplied
through the basis set file. There are multiple choices for the Basis Set, Section x 
will give an overview of these choices. Likewise the pseudopotentials for using with
plane waves must also be supplied.

The exchange-correlation (XC) functional within DFT contains approximations which make 
it a source of inaccuracy in a DFT calculation. This stems from the fact that 
the exact functionals for exchange and correlation are not known.
There are many choices of XC functional,
with different levels of accuracy (see Jacobs ladder). However greater accuracy 
usually comes with more computational cost.  Section x will outline the available options
for XC functionals in CP2K and how to use them.

In Quickstep, a self-consistant (SCF) calculation is performed in order to find the ground 
state energy of the system (with the atom positions fixed).
This involves performing a number of SCF steps
where in each step the potential is calculated from the electronic density and 
then this is used to construct a new electron density by solving the KS equations 
(this density is then used in the next SCF step). The SCF converges when the
required tolarence for self-consistency is met. This will correspond to ground
state energy minimum. As this calculation is self-consistent it will depend
highly on the starting electronic density, a good starting density will allow
the calculation to converge faster. If the SCF has not converged after it has
exceeded the maximum number of steps (set by MAX_SCF) the SCF calculation will 
terminate and print the warning message: "SCF has not converged".

The SCF calculation involves inner and outer loops. If the inner SCF loop does not
converge in the desired number of steps (set in MAX_SCF) then the inner loop will exit in order to
prevent wasting time heading in the wrong direction. The preconditioner is
recalculated and then a new inner loop SCF begins, with the number of outer 
steps incremented by one. This will hopefully stand a better chance of converging
than the previous inner cycle. This process will be repeated for a number of outer
steps (set in the &OUTER_SCF section). After this the SCF calculation is
terminated. Hence, the maximum number of SCF steps attempted is given as the product
of the inner SCF steps and the outer SCF steps.





---------------------------
A basic QM input
---------------------------

A basic DFT input section for a calculation using the Gaussian Plane Wave (GPW) method is shown below.
This is designed to act as a rough guide for how to build your DFT section, and contains some example
parameter settings with descriptions in the comments. However it should not be taken as an example set
up for your system, and should certainly not be used without first considering the choices for the
important paramters outlined below.

.. Examples for using a Semi-emperical method (SE) and the Tight Binding method (TDFT) are provided here:

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
      &OT  T                           ! Use the orbital transform method (T: true)
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
to one of the basis sets for the given element within the basis set file.
The q number proceeding the basis set in the identifer gives the number of 
valence electrons. It depends on the element, for example H:1, C:4, O:6, N:5.

Basis set files are provided within the /data directory in CP2K (link).
If your install of CP2K  has been built correctly then
the files within this directory should be automatically included, so there is no
need to provide these in you working directory. 

The GTH basis sets are usually recommended in CP2K, there also exist the molecular optimisted (MOLOPT) GTH
basis set. 
Some common options for basis
sets and their location within the basis set files are shown in the table below. 

+--------------------------------------------------+--------------------------------+--------------------------------------+-------------------------------------------------+
| Description                                      | GTH (cp2k_root/data/BASIS_SET) | MOLOPT (cp2k_root/data/BASIS_MOLOPT) | Comments                                        |
+==================================================+================================+======================================+=================================================+
| Single-zeta valence                              | SZV-GTH                        | SZV-MOLOPT-GTH                       | Use only for testing                            |
+--------------------------------------------------+--------------------------------+--------------------------------------+-------------------------------------------------+
| Double-zeta valence polarised                    | DZVP-GTH                       | DZVP-MOLOPT-GTH                      | A good choice, available for most elements      |
+--------------------------------------------------+--------------------------------+--------------------------------------+-------------------------------------------------+
| Triple-zeta valence polarised                    | TZVP-GTH                       | TZVP-MOLOPT-GTH                      | More accurate than DZVP                         |
+--------------------------------------------------+--------------------------------+--------------------------------------+-------------------------------------------------+
| Triple-zeta valence 2x polarisation functions    | TZV2P-GTH                      | TZV2P-MOLOPT-GTH	                   | More accurate still, may not have some elements |
+--------------------------------------------------+--------------------------------+--------------------------------------+-------------------------------------------------+
| Quadrupal-zeta valence 2x polarisation functions | QZV2P-GTH                      | QZV2P-MOLOPT-GTH	                   | Most accurate but least availablity             |
+--------------------------------------------------+--------------------------------+--------------------------------------+-------------------------------------------------+


The choice of basis will depend on the accuracy required, and whether it is available for the elements in your system. 
More accurate basis sets will increase the runtime, and may not be available for some elements e.g. metal ions.

The error in due to the basis set is  smaller than the XC functional so chosing a large basis may not be sensible 
unless using a very accurate XC functional.

Using the DZVP basis set is usually a good choice. If you would like to explore more accurate options
then you may consider checking the convergence of your basis set by plotting the number of independent orbital functions vs. the energy.


---------------------
XC functionals
---------------------

Overview
--------

The exchange-correlation (XC) functional within DFT contains approximations which make 
it a source of inaccuracy in a DFT calculation. Choosing an XC functional is therefore
an important consideration, it has the potential to be the largest source of error in
a DFT calculation. 

There are many choices of XC functional,
with different levels of accuracy, however increased accuracy usually requires longer run time to solve 
so this is a trade-off that you will have to consider when picking your functional. 

The XC functional is set up is described in the XC section of the CP2K input. You will
also want to consider your choice of pseudopotential in combination with your XC functional,
some pseudopotentials have been optimised for given XC functionals.

The table below lists the XC functional options available in CP2K from least to
most accurate, and gives a overview of each option.

+----------------+-------------------------------------+-----------------+---------------------------------------------------------------------------------------------------+
| Type           | Description                         | CP2K examples   | Comments                                                                                          |
+================+=====================================+=================+===================================================================================================+
| LDA            | local density approximation	       | PADE, PW92      | fast but not accurate                                                                             |
+----------------+-------------------------------------+-----------------+---------------------------------------------------------------------------------------------------+
| GGA            | generalised gradient approximation  | BLYP, PBE, PW91 | usually a good choice if you are not worried about being very accurate or have a large QM region  |
+----------------+-------------------------------------+-----------------+---------------------------------------------------------------------------------------------------+
| metaGGA        | metaGGA (higher order terms)        | TPSS            | Available through Libxc library                                                                   |
+----------------+-------------------------------------+-----------------+---------------------------------------------------------------------------------------------------+
| Hybrid         | Hartree Fock exchange + GGA method  | B3LYP, PBE0     | More accurate,                                                                                    |
+----------------+-------------------------------------+-----------------+---------------------------------------------------------------------------------------------------+
| Double hybrid	 | HFX + PT2 correlation + GGA methods | B2PYLP          | Most accurate, can requires many times more time than GGA etc.                                    |
+----------------+-------------------------------------+-----------------+---------------------------------------------------------------------------------------------------+

Empirical vs. non expirical
---------------------------




LDA
---

The local density approximation is one the simplist approximations for the XC functional.
It assumes that the functional depends only on the density at one point, i.e the density
is assumed to be smooth in space.

An example for using the PADE LDA method is shown below. The functional needs to be specified
in the XC_FUNCTIONAL section, and the complementary GTH-PADE pseudopotentials should be used.

.. code-block::

    &XC
      &XC_FUNCTIONAL PADE
      &END XC_FUNCTIONAL
    &END XC


GGA
---

The generalised gradient approximation is an improvement on the LDA which takes into account the 
gradient of the density, as well as the density at one point.

Using the GGA in CP2K is simular to using the LDA. It requires specifying the functional 
and using the complementary  pseudopotentials (in this case GTH_PBE).

.. code-block::

    &XC
      &XC_FUNCTIONAL PBE
      &END XC_FUNCTIONAL
    &END XC

metaGGA
-------



Hybrid methods
--------------

Hybrid methods cacultate a portoion fo the the exchange functional using the exact Hartree Fock theory.
The rest of the exchange and correlation functions is calcaulated with other methods, typically GGA or LDA.
The XC section of the CP2K input has a HF section is used for the hartree fock set up.

Two commonly used hybrid methods dicussed here are B3LYP and PBE0.

**PBE0**

In the PBE0 functional the exchange is comprised of 75% of the PBE exchange and 25% of the HF exchange.
The correlation energy is full PBE.

.. math::

    E^{PBE0}_{XC} = \frac{1}{4} E_X^{HF} + \frac{3}{4} E_X^{PBE} + E_C^{PBE}

In CP2K to use the PBE0 functional the XC section of the input file should be
configured as follows:

.. code-block::

    &XC
       &XC_FUNCTIONAL
       &PBE
         SCALE_X 0.75         ! 75% GGA exchange
         SCALE_C 1.0          ! 100% GGA correlation
       &END PBE
      &END XC_FUNCTIONAL
      &HF
        FRACTION 0.25         ! 25 % HF exchange
        &SCREENING        
          EPS_SCHWARZ 1.0E-6  ! Important to improve scaling
        &END
        &MEMORY
          MAX_MEMORY 1500     ! In MB per MPI rank
        &END
    &END


**B3LYP**

The B3BLYP functional which stands for (Becke, 3-parameter, Lee–Yang–Parr
It makes use of the HF exchange and GGA functionals for the exchange and correlation
(in particular the Becke 88 exchange functional and the LYP correlation functional).

.. math::

    E^{B3LYP}_{XC} = E_X^{LDA} + a_0(E_X^{HF} - E_X^{LDA}) + a_x(E_X^{GGA} - E_X^{LDA}) + E_C^{LDA} + a_c(E_C^{GGA} - E_C^{LDA})
    
.. code-block::

   &XC
      &XC_FUNCTIONAL
         &LYP
            SCALE_C 0.81          ! 81% LYP correlation
         &END 
         &BECKE88
            SCALE_X 0.72          ! 72% Becke88 exchange
         &END
         &VWN
            FUNCTIONAL_TYPE VWN3
            SCALE_C 0.19          ! 19% LDA correlation
         &END 
         &XALPHA
            SCALE_X 0.08          ! 8%  LDA exchange
         &END 
      &END XC_FUNCTIONAL
      &HF
         FRACTION 0.20            ! 20% HF exchange
         &SCREENING
            EPS_SCHWARZ 1.0E-10   ! Improves scaling
         &END 
         &MEMORY
            MAX_MEMORY  1500     ! In MB per MPI rank
         &END
      &END
   &END XC
 

Double-hybrid methods
---------------------

B2PLYP

Dispersion corrections
----------------------

Higher order methods
--------------------

---------------------
Puesdopotentials
---------------------


------------------------
Important QM parameters
------------------------

CUTOFF
------

The CUTOFF parameter sets 

REL_CUTOFF
----------

NGRID
-----

EPS_SCF
-------

COMMENSURATE
------------

----------------------------
Converging the CUTOFF/REL_CUTOFF
----------------------------

-----------------
Troubleshooting
-----------------

My energy is positive
---------------------

SCF does not converge
---------------------

Some other CP2K error messages
------------------------------

