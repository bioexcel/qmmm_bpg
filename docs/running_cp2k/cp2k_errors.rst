==========================
CP2K Troubleshooting
==========================

-----------------------
Abort messages
-----------------------

Abort messages appear when something has occured that causes CP2K to exit. They 
will look like this and contain a message about what has caused the abort, and 
what can be done to fix it.


.. code-block :: none

 *******************************************************************************
 *   ___                                                                       *
 *  /   \                                                                      *
 * [ABORT]                                                                     *
 *  \___/                         Error messsge here                           *
 *    |                                                                        *
 *  O/|                                                                        *
 * /| |                                                                        *
 * / \                                                                         *
 *******************************************************************************


GEOMETRY wrong or EMAX_SPLINE too small!
----------------------------------------

This is usually means there is a problem with the MM forcefield or the geometry of your system.
You can first try to increase ``EMAX_SPLINE`` to verify if this was the issue.
If this does not fix the error then you should double check the correctness of the MM forcefield and
geometries.


KIND not found
---------------

You may get an error message from CP2K saying ``"Unknown element for KIND"``. This is becasue CP2K only expects
proper element symbols in the coordinate and forcefield files. The work around for this is
to let CP2K know what element the symbol should correspond to. This is done by adding a new ``KIND`` section
in the ``SUBSYS`` section for the unknown symbol, or by specifying the corresponding element symbol directly
in the ``pdb`` coordinates file.

Use the LSD option for an odd number of electrons
-------------------------------------------------

You will also get this message if the total number of QM electrons is odd. First you
should check that the charge of you QM region is correct and adjust ``DFT&CHARGE`` and 
``DFT&MULTIPLICITY`` if necessary (see section QM_treatmenent).

If indeed the charge is odd then you will need to do a spin polarised calculation 
by specifying ``LSD`` under ``DFT``. This will take up to twice the time for a non-spin polarised 
calculation so it should be avoided unless really necessary.

Cholesky decomposition failed. Matrix ill conditioned ?
-------------------------------------------------------

Check that the multiplicity is correct. The default is 1 (singlet) for an even
number and 2 (doublet) for an odd number of electrons.

Check that employed pseudopotentials are correct e.g. you are specifying the right
pseudopotentials set for the elements in the QM system.

The requested basis set for element was not found in the basis set files
--------------------------------------------------------------------------

This message will also report the name of the element.
Check that the basis set file name in the ``SUBSYS`` section is the correct one.

Error in QM/MM Connectivity
---------------------------

.. code-block:: none

 ERROR in the QM/MM connectivity. A QM/MM LINK was detected but
 no LINK section was provided in the Input file!
 This very probably can be identified as an error in the specified QM
 indexes or in a missing LINK section. Check your structure!

This error is fairly self explanatory. It means that the QM region has not been 
properly defined based on the connectivity between atoms. You are either missing QM
atoms from the QM region in ``QMMM&QM_KIND`` or a QM to MM LINK section is missing.

------------------------
CPASSERT failed messages
------------------------

This error is produced when a check within the CP2K code fails. It can be difficult to diagnose
as the error is not always very specific and ways to solve it are not given.
Sometimes a filename and line number from where the error was reported in the code is printed.
This can help give some idea about what the error might be.


Some common causes of CPASSERT failure and how to identify them are listed below.

COMMENSURATE not used
---------------------

Not turning on ``COMMENSURATE`` in the ``MGRID`` section will cause a CPASSERT error. This 
error will reference ``pw_spline_utils.F``.

``pw/pw_spline_utils.F:1086``

---------------------
Runtime issues
---------------------

Runtime issue are those that do not produce an error or cause the calculation to 
crash. Sometimes a warning message may be printed in the output file to highlight that
something is not behaving as expected, or the calculation might produce 
unexpected results. 

Simulation fails or gives strange results
-----------------------------------------

Providing you have used a sensible QM setup with sufficiently large enough cutoffs then the error has usually to do with the setup of your 
system. If running a periodic calculation check that the CELL boundaries are large enough to separate the periodic images.
Also check the initial atomic coordinates are sensible by visualising your system. 

If this looks correct then consider simpifying 
you input, starting with the most simple settings, and choices for basis sets and functionals. If the QM/MM simulation fails then
may want to try running a simple MM calcaultion first (``RUN_TYPE FIST)`` to check the geometries, and then slowly increase the complexity
adding in ``QM`` and ``QMMM`` sections.



SCF does not converge
---------------------

If the energies are rapidly varying then it is likely that the SCF is failing to converge. This will be reported in the cp2k output
with the message ``"WARNING SCF has not converged"``. You can quickly double whether the SCF has failed to converge by using grep to 
search your output for this message:

``grep 'WARNING SCF' output-file.log``

If this occurs then the easiest variables to change to try and fix this are the ``MAX_SCF`` and ``EPS_SCF``.

Some things to try are listed below:

* Check ``OUTER_SCF&EPS_SCF <= EPS_SCF``. If not decrease the outer ``EPS_SCF``.
* Increase the number of SCF loops with ``OUTER_SCF&MAX_SCF``.
* Increase the number of inner SCF steps with ``MAX_SCF``.
* Change the OT minimiser to CG.
* Check your geometry again.
* If running MD consider decreasing your timestep.


