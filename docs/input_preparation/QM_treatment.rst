==============================
 QM treatment
==============================

This section of the guide will focus on setting up the QM parameters in a QM/MM simulation.


It will guide on how to use various options within CP2K, and give some general advice but
it will not give any information about what specific choices (e.g. basis sets and XC functionals)
are best for your system.

--------------------------------------
Mechanics of a DFT calculation in CP2K
--------------------------------------

CP2K uses the Quickstep (QS) method when performing a QM calculation.
Quickstep is the part of the code in CP2K devoted to solve the electronic
structure problem in order to get energies in the QM region and forces
on the QM atoms. The default electronic structure method is the
self-consistent Kohn-Sham Density Functional Theory (DFT). Alternatively, within Quickstep
there is also the option to use Semi-empirical (SE), Tight-Binding DFT (DFTB),
and Gaussian Plane Waves (GPW) methods, among many others. The GPW method is a 
key part of CP2K, which involves using a mixed basis of atom centred Gaussian
orbitals and plane waves (regular grids) to improve on the performance compared
to other QM methods. You can find information about GPW here: https://www.cp2k.org/quickstep

Basis functions described in the Gaussian type orbitals for each element are supplied
through the basis set file. For the basis set, multiple choices are available. The :ref:`ref_basis_sets`
section of the guide will give an overview of these choices. Likewise the pseudopotentials for using with
plane waves must also be supplied.

The exchange-correlation (XC) functional within DFT contains approximations which make 
it a source of inaccuracy in a DFT calculation. This stems from the fact that 
the exact functionals for exchange and correlation are not known.
There are many choices of XC functional,
with different levels of accuracy (see Jacobs ladder). However greater accuracy 
usually comes with more computational cost.  The :ref:`ref_xc` section will outline the available options
for XC functionals in CP2K and how to use them.

In Quickstep, a self-consistant (SCF) calculation is performed in order to find the ground 
state energy of the system (with the atom positions fixed).
This involves performing a number of SCF steps
where in each step the potential is calculated from the electronic density and 
then this is used to construct a new electron density by solving the KS equations 
(this density is then used in the next SCF step). The SCF converges when the
required tolarence for self-consistency is met. The electronic state found at the
end of a converged SCF calculation represents the best prediction of the employed
method for the electronic ground state energy minimum.  As this calculation is self-consistent,
its convergence properties will highly depend on the starting electronic density. Therefore,
a good starting electronic density will allow the calculation to converge faster.
If the SCF has not converged after it has
exceeded the maximum number of steps (set by MAX_SCF) the SCF calculation will 
terminate and print the warning message: "SCF has not converged". Information on 
how to overcome the issue of a non-converging SCF calculation can be found under :ref:`ref_troubleshooting`.

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
important parameters outlined below.

.. Examples for using a Semi-emperical method (SE) and the Tight Binding method (TDFT) are provided here:

.. code-block:: none

  &DFT                                 ! CP2K input block for DFT
    CHARGE 0                               ! charge of the QM region
    MULTIPLICITY 1                         ! multiplicity of the QM region
    BASIS_SET_FILE_NAME  bs_filename       ! name of the basis set filename
    POTENTIAL_FILE_NAME  pot_filename      ! name of the potential filename

    &MGRID
      CUTOFF 300                           ! planewave cutoff
      REL_CUTOFF 50                        ! relative cutoff for gaussian grid
      COMMENSURATE                         ! If grids are commensurate (always true for QM/MM)
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

.. code-block:: none

  &SUBSYS
    &KIND H
      ELEMENT H
      BASIS_SET bs_identifier
      POTENTIAL pot_identifier
    &END KIND
  &END SUBSYS
 
.. _ref_basis_sets:

------------
Basis sets
------------

The basis set for each element can be changed by editing the bs_filename within the DFT section, and the bs_identifier 
in the KIND section of that element within the SUBSYS section. The bs_identifier should correspond
to one of the basis sets for the given element within the basis set file.
The q number proceeding the basis set in the identifer gives the number of 
valence electrons. It depends on the element, for example H:1, C:4, O:6, N:5.

Basis set files are provided within the /data directory of the CP2K source code
(https://github.com/cp2k/cp2k/tree/master/data).
If your installation of CP2K  has been built correctly then
the files within this directory should be automatically included, so there is no
need to copy these file to your working directory. 

The GTH basis sets are usually recommended in CP2K, there also exists a molecular optimisted (MOLOPT) GTH
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


The choice of basis depends on the accuracy required, and whether it is available for the elements in your system. 
More accurate basis sets will increase the run time of the simulation, but may not be available for some elements e.g. metal ions.

The error due to the basis set in general is smaller than the error associated to the XC functional. Therefore, chosing a large basis set may not be sensible 
unless you require a very accurate calculation and you are employing an accurate XC functional.

Using the DZVP basis set is usually a good compromise. If you would like to explore more accurate options
then you may consider checking the convergence of your basis set by plotting the number of independent orbital functions vs. the energy.

.. _ref_xc:

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
with different levels of accuracy, however increased accuracy usually requires longer run time,
so this is a trade-off that you will have to consider when picking your functional. 

The XC functional is setup is described in the XC section of the CP2K input. The choice of
the functionals could also depend on the availability of the corresponding pseudopotentials.
In fact, each pseudopotential is built using a specific XC functional and it should be used
only in combination with that XC functional. Usually, the name of the pseudopotential file 
reports explicitly the XC functional used to build it.

The table below lists the XC functional types available in CP2K from least to
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




LDA
---

The local density approximation is one of the simplest approximations for the XC functional.
It assumes that the functional depends only on the density at one point, i.e the density
is assumed to be smooth in space.  Such an approximation is rather crude and often provide
inaccurate results for some properties.

 An example of how to setup the PADE LDA method in the CP2K input file is shown below. 
 The functional needs to be specified in the XC_FUNCTIONAL section, 
 and the corresponding GTH-PADE pseudopotentials should be used.

.. code-block:: none

    &XC
      &XC_FUNCTIONAL PADE
      &END XC_FUNCTIONAL
    &END XC



GGA
---

The generalised gradient approximation (GGA) is an improvement on the LDA which takes into account the 
gradient of the density, as well as the density at one point.

Using the GGA in CP2K is similar to using the LDA. It requires specifying the functional 
and using the complementary pseudopotentials (which in this case would be GTH_PBE).

.. code-block:: none

    &XC
      &XC_FUNCTIONAL PBE
      &END XC_FUNCTIONAL
    &END XC

Using a GGA functional is usually a good starting point for running a QM calculation. It is not
computationally expensive and it is simple to set up in CP2K. 

**BLYP or PBE?**

BLYP and PBE are the most commonly used GGA functionals. The main difference between them is
that PBE is non-empirical i.e. the parameters are based on theoretical consideration and calculations,
while BLYP is partially-empirical because some parameters were obtained via emperical fittings.
As a result PBE gives rather accurate results 
for a wide range of systems, whereas BLYP can be more accurate than PBE for some particular systems.
This consideration also holds for the hybrid methods PBE0 and B3LYP which are derived from their GGA
counterparts PBE and BLYP, respectively (see below).
If BLYP/B3LYP are not widely used in your research area then it may be prudent to use PBE or PBE0 instead.



metaGGA
-------

The metaGGA builds upon the GGA methods by assuming the functional also depends on
then non-interacting kinetic energy density, in addition to the electron density and its 
gradient. To use metaGGA methods in CP2K the libxc library is used, and therefore your
version of CP2K needs to be built with this library enabled. An example of the XC
section for using the metaGGA is shown below (here the oTPSS-D functional has
been used (http://doi.org/10.1021/ct900489g) ).


.. code-block:: none

   &XC 
      &XC_FUNCTIONAL
         &LIBXC T                        ! use libxc library
          FUNCTIONAL MGGA_XC_OTPSS_D     ! oTPSS-D functional
         &END LIBXC
      &END XC_FUNCTIONAL
   &END XC


There are a variety of metaGGA method available through libxc, details of these 
can be found here: https://www.tddft.org/programs/libxc/functionals/ (note that 
functional availablity is dependent on the version of libxc used).

Hybrid methods
--------------

Hybrid methods calculate a portion of the the exchange functional using exact Hartree Fock theory.
The rest of the exchange and correlation functions is calcaulated with other methods, typically GGA or LDA.
Within the XC section of the CP2K input the HF section is used for the Hartree Fock exchange setup.
Two commonly used hybrid methods dicussed here are B3LYP and PBE0.

**PBE0**

In the PBE0 functional the exchange is comprised of 75% of the PBE exchange and 25% of the HF exchange.
The correlation energy is entirely PBE.

.. math::

    E^{PBE0}_{XC} = \frac{1}{4} E_X^{HF} + \frac{3}{4} E_X^{PBE} + E_C^{PBE}

In CP2K to use the PBE0 functional the XC section of the input file should be
configured as follows:

.. code-block:: none

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

The B3LYP functional stands for - Becke, 3-parameter, Lee–Yang–Parr.
It makes use of the HF exchange and GGA functionals for the exchange and correlation
(in particular the Becke 88 exchange functional and the LYP correlation functional).
Three parameters are used in its description:

.. math::

    E^{B3LYP}_{XC} = E_X^{LDA} + a_0(E_X^{HF} - E_X^{LDA}) + a_x(E_X^{GGA} - E_X^{LDA}) + E_C^{LDA} + a_c(E_C^{GGA} - E_C^{LDA})
    
where :math:`a_0` = 0.2, :math:`a_x` = 0.72 and :math:`a_c` = 0.81.
To use B3LYP in CP2K the XC section of the input file should be
configured as follows:

.. code-block:: none

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
 
---------------------
Pseudopotentials
---------------------

As mentioned before, each pseudopotential is built using a specific XC functional
and it should be used only in combination with that XC functional. For example the GTH-PBE
pseudopotential should be used with the PBE XC functional.

----------------------
Dispersion corrections
----------------------

DFT is known to underestimate van der Waals forces between atoms. Empirical dispersion
corrections can be used in combination with XC functionals to improve the description of
van der Waals forces, which can play an important role in protein
systems.

In CP2K three different dispersion options are available, DFT-D2, DFT-D3 and DFT-D3(BJ).
All three of these methods involve adding
an extra dispersion term to the energy density functional, e.g.

.. math::

 E_{tot} = E_{DFT} + E_{disp}

The DFT-D3 method offers improvements on the DFT-D2 method,
and the DFT-D3(BJ) method adds Becke-Jonson damping to the dispersion energy.

To use a dispersion correction the 
vdW_POTENTIAL section is added inside the XC_FUNCTIONAL section. An example of
the vdW_POTENTIAL section is shown below:

.. code-block:: none

  &vdW_POTENTIAL
     DISPERSION_FUNCTIONAL PAIR_POTENTIAL     ! usually set to pair_potential
     &PAIR_POTENTIAL
        TYPE vdw-type                         ! VDW type (DFT-D2, DFT-D3 or DFT-D3(BJ)
        PARAMETER_FILE_NAME dftd3.dat         ! required for DFT-D3 and DFT-D3(BJ)
        REFERENCE_FUNCTIONAL xc_type          ! the reference xc functional e.g. PBE, B3LYP    
      &END PAIR_POTENTIAL
  &END vdW_POTENTIAL





------------------------------
Important QM input parameters
------------------------------

CHARGE
------

This is used to set the charge of the QM part of the system.

MULTIPLICITY
------------

The multiplicity should be set to twice the total spin plus one. 
If set to 0 (the default) this will be 1 for an even number of electrons and 2 for an odd 
number of electrons. 

CUTOFF
------

The CUTOFF parameter sets the planewave cutoff (given in units of Ry). It is an important
parameter in a QM calculation, and choosing a too small cutoff can result in large inaccuracies 
in the energy. A larger cutoff is usually more accurate as the planewave grid becomes finer,
however at a certain point increasing the 
cutoff would no longer make any difference to the energy, but would increase the computational cost.

Before doing a production run it is important to converge the cutoff. This process is
described in detail here: https://www.cp2k.org/howto:converging_cutoff .
It essentially involves tracking the energy as the cutoff is varied
and then selecting a cutoff large enough such that the energy reaches convergence. The correct value
of the cutoff depends on the basis set, the pseudopotentals, the XC functional and the system itself.
Therefore, the above convergence test must be performed whenever one of these elements is changed.

REL_CUTOFF
----------

The REL_CUTOFF is similar to the CUTOFF and sets the planewave cutoff of a reference grid
covered by a Gaussian function with unit standard deviation. This parameter is important to map Gaussian functions on a grid.
Converging this parameter is also covered in this guide: https://www.cp2k.org/howto:converging_cutoff.

COMMENSURATE
------------

COMMENSURATE is a logical option which specifies if the grids should be commensurate or not. In a QM/MM
calculation this must be set to true.

EPS_DEFAULT
-----------

This parameter provides an easy way to set all the EPS_xxx parameters to
values such that the energy will be correct up to this value. 
The default value for this is 1.0E-10. Decreasing this value will slightly increase the 
accuracy of the energy, but will also increase significantly the run time.

EPS_SCF
-------

This sets the target accuracy for the SCF convergence. The SCF will be converged when the energy change between two SCF
steps is less than this value. The default for this value is 1.0E-5. It is possible to set different values for the inner
and outer SCF loops, however the EPS_SCF of the outer SCF must be smaller than or equal to EPS_SCF of the inner loop. In fact
the EPS_SCF of the inner loop determines the value that can be reached in the outer loop.

MAX_SCF
-------

In the main SCF section of the input this keyword sets the maximum number of SCF iterations to be performed in the inner SCF loop.
In the OUTER_SCF section this keyword sets the maximum number of outer loops. The total number of SCF steps will be at maximum the product
of the MAX_SCF for the inner SCF loop and MAX_SCF for the outer SCF loop.

.. _ref_troubleshooting:

-----------------
Troubleshooting
-----------------

Simulation fails or gives strange results
-----------------------------------------

Providing that you have used a sensible QM setup with a sufficiently large cutoff then
the error is usually related to the setup of your system. When running a calculation with periodic boundary 
conditions check that the CELL boundaries are large enough to keep the periodic
images sufficiently separated. A convergence test for the CELL size can be crucial in this case.
Also check the initial atomic coordinates are sensible by visualising your system. 

If the initial coordinates look reasonable then consider simplifying 
your input, starting with the most simple settings, including basis sets and functionals. If the QM/MM simulation fails then
may want to try running a simple MM calcaultion first (RUN_TYPE FIST) to check the geometries, and then slowly increase the complexity
adding in QM and QM/MM sections.

SCF does not converge
---------------------

If during the SCF calculation the energies varies rapidly then it is likely that
the SCF will not converge. This will be reported in the CP2K output with the message 
"WARNING SCF has not converged. You can quickly verify if the SCF has failed to converge by 
looking for this text in your output file:

``grep 'WARNING SCF' output-file.log``

If this occurs then the easiest parameters to change to try to tune in order to reach SCF convergence are the MAX_SCF and EPS_SCF.

Some things to try are listed below:

* Check OUTER_SCF&EPS_SCF <= EPS_SCF. If not decrease the outer EPS_SCF.
* Increase the number of SCF loops with OUTER_SCF&MAX_SCF.
* Increase the number of inner SCF steps with MAX_SCF.
* Change the OT minimizer to CG.
* Check again your geometry.
* If running MD consider decreasing your timestep.



