=================
MM setup
=================

An example CP2K input for a simple single energy MM calculation is shown below.
Note that the RUN_TYPE is set to ENERGY and the METHOD in the FORCE_EVAL section
is set to FIST for an MM run. This stands for Frontiers in Simulation Technology which is the 
method used in CP2K MM calculations.



.. parsed-literal:: 

  &GLOBAL
     PROJECT project_name
     RUN_TYPE ENERGY
     PRINT_LEVEL LOW
  &END GLOBAL
  &FORCE_EVAL
     METHOD FIST                       ! Do MM
     &MM
       &FORCEFIELD
       PARMTYPE AMBER                  ! use the Amber forcefield type (see also :ref:`ref_ffield`)
       DO_NONBONDED .TRUE.             ! short range non bonded interactions
       PARM_FILE_NAME ff_name          ! forcefield filename
       &SPLINE
          :ref:`ref_emax_spline` 1.0E14           ! max spline
          :ref:`ref_rcut_nb` [angstrom] 12        ! Cutoff radius for nonbonded interactions
       &END SPLINE
       &END FORCEFIELD
       &POISSON
          &EWALD
             :ref:`ref_ewald_type` SPME           ! recommended ewald type
             :ref:`ref_gmax` 70                   ! number of grid points 1 per angstrom in each direction
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
           CONN_FILE_FORMAT AMBER      ! use the Amber format (see also :ref:`ref_ffield`)
           CONN_FILE_NAME ff_name      ! amber forcefield filename
           COORD_FILE_FORMAT PDB       ! coords in pdb formant (see also :ref:`ref_coords`)
           COORD_FILE_NAME coord_name  ! coordinate filename
        &END TOPOLOGY
     &SUBSYS
  &END FORCE_EVAL



The MM and SUBSYS sections of FORCE_EVAL are required for this calculation. The MM section will contain 
all the parameters for MM such as the forcefield, and the poisson and spline information.
The subsys section contains the systems topology information
such as the atomic coordinates, the cell size and the connectivity.

.. _ref_ffield:

----------------------------
MM forcefields types in CP2K
----------------------------

Amber
-----

If using an Amber forcefield the .prmtop file should be used as the forcefield file
``PARM_FILE_NAME`` in the FORCEFIELD section and as the connectivity file ``CONN_FILE_NAME`` 
in the TOPOLOGY section. Both the ``PARMTYPE`` and ``CONN_FILE_FORMAT`` should be set to ``AMBER``.

CHARMM
-------

The CHARMM force field  (i.e. a .prm file) can be used by setting ``PARMTYPE`` to CHM. The 
psf CHARMM connectivity file is used with ``CONN_FILE_FORMAT`` set to PSF.

GROMOS
------

The GROMOS 96 format can be used by specifying ``PARMTYPE`` G96.



---------------------
Connectivity formats
---------------------

- **AMBER** - Use AMBER topology file .prmtop or .top
- **G87** - Use GROMOS G87 topology file.
- **G96** - Use GROMOS G96 topology file.
- **PSF** - Use a CHARMM PSF file.
- **UPSF** - Use an unformatted PSF file.

.. _ref_coords:

----------------------------
Coordinate formats
----------------------------

The atomic coordinates are supplied in the topology section. The following different file 
types are allowed. 

- **CIF** - Coordinates provided through a CIF (Crystallographic Information File) file format
- **CRD** - Coordinates provided through an AMBER file format e.g. .inpcrd .crd
- **G96** - Coordinates provided through a GROMOS96 file format
- **PDB** - Coordinates provided through a PDB file format
- **XTL** - Coordinates provided through a XTL (MSI native) file format
- **XYZ** - Coordinates provided through an XYZ file format



Note that even if your coordinates file contains information about the 
box dimensions these should be listed in the cp2k input in the CELL section.



-----------------------------
Important MM input parameters
-----------------------------


.. _ref_emax_spline:

EMAX_SPLINE
-----------

Specifies the maximum value of the potential up to which splines will be constructed

.. _ref_rcut_nb:

RCUT_NB
-------

Cutoff radius for nonbonded interactions. This value overrides the value specified 
in the potential definition and is global for all potentials.

.. _ref_ewald_type:

EWALD_TYPE
----------

EWALD is the standard non-fft based ewald
NONE standard real-space coulomb potential is computed together with the non-bonded contributions
PME is the particle mesh using fft interpolation
SPME is the smooth particle mesh using beta-Euler splines (recommended)

.. _ref_gmax:

GMAX
----

Number of grid points (SPME and EWALD). Supply a single variable N for all three dimensions or Nx, Ny, Nz 
for individiual dimensions. One point per Angstrom is common, however this may cause the calculation to be
slow for larger QMMM cells.

---------------
Troubleshooting
---------------


GEOMETRY wrong or EMAX_SPLINE too small!
----------------------------------------

This is usually means there is a problem with the MM forcefield or the geometry of your system.


KIND not found
---------------

You may get an error message from CP2K saying "Unknown element for KIND". This is becasue CP2K only expects
proper element symbols in the coordinate and force field files. The work around for this is
to let CP2K know what element the symbol should correspond to. This is done by adding it as its own KIND section
in the SUBSYS section, or by specifying elements in the PDB coordinates file.
