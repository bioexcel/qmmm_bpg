=================
CP2K MM setup
=================

An example CP2K input for a simple single energy MM calculation is shown below.
Note that the ``RUN_TYPE`` is set to ``ENERGY`` and the ``METHOD`` in the ``FORCE_EVAL`` section
is set to ``FIST`` for an MM run. This stands for Frontiers in Simulation Technology which is the 
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
          :ref:`ref_emax_spline` 1.0E04           ! max spline
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



The ``MM`` and ``SUBSYS`` sections of ``FORCE_EVAL`` are required for this calculation. The ``MM`` section contains 
all the parameters related to the atomic interations such as the forcefield, and Poisson solver and splines
used in the non-bonded interations.
The ``SUBSYS`` section contains the systems topology information
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

The CHARMM force field  (i.e. a ``.prm`` file) can be used by setting ``PARMTYPE`` to CHM. The 
psf CHARMM connectivity file is used with ``CONN_FILE_FORMAT`` set to PSF. 
Example usage can be found here: https://www.cp2k.org/exercises:2015_cecam_tutorial:forcefields

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

This parameter specifies the type of non-bonded long-range interaction method should be used in the calculation.
The following options are available.

- **NONE** - standard real-space coulomb potential is computed together with the non-bonded contributions
- **EWALD** - standard non-fft based ewald
- **PME** - particle mesh using fft interpolation
- **SPME** - smooth particle mesh using beta-Euler splines (recommended)

.. _ref_gmax:

GMAX
----

Number of grid points (SPME and EWALD). Supply a single value N for all three dimensions or Nx, Ny, Nz 
for individiual dimensions. One grid point per Angstrom is a typical chocie, however such a value may 
cause the calculation to become too slow for large cells.

---------------
Troubleshooting
---------------


GEOMETRY wrong or EMAX_SPLINE too small!
----------------------------------------

This is usually means there is a problem with the MM forcefield or the geometry of your system.


KIND not found
---------------

You may get an error message from CP2K saying ``"Unknown element for KIND"``. This happens when a symbol
that does not match a proper element is found in the coordinate and force field files. The workaround
for this is to let CP2K know what element the offended symbol should correspond to. This is done by
adding in the ``SUBSYS`` section a new ``KIND`` section for the novel symbol where to specify the element
via the keyword ``ELEMENT``. Alternatively, one can specify the element symbol in the ``PDB`` coordinate file.
