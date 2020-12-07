==========================
Selecting QM atoms
==========================


-----------------
Initial selection
-----------------

The starting point for deciding which atoms should be treated at a quantum level is the ligand or the
region of interest in your system (such as a metal ion). The size of the ligand will usually 
dictate whether you can treat the whole ligend, a subsection of the ligand, 
or the ligand plus some neighbouring residues with QM. As previously mentioned, ideally the 
number of QM atoms should not be too large (over 200 QM atoms) as this will become too 
computationally demanding. However we want to be able to include all the regions 
involved in the chemical reaction as much as possible.

A good approach for selecting the QM region is therefore to select the entire ligand plus
some residues nearby, say within x :math:`\AA` of the ligand. The size you consider for this region
around the ligand will depend on the number of atoms in your ligand. If you have a ligand of
50 or more atoms then you might only include residues within 2-3 Å in order to limit 
the overall size of the QM region. For a smaller ligand you may be able to include more atoms around it
(up to approximately 6 :math:`\AA`).


Creating the QM atoms list for CP2K
-----------------------------------

Creating a list of  QM atoms that can be read by CP2K can be done using vmd. (https://www.ks.uiuc.edu/Research/vmd/)

1) Open the entire system pdb file with ``vmd``.

.. code-block:: none

 vmd system.pdb

2) Go to File >> Save coordinates

3) Use Selected Atoms to save a pdb containing the desired QM atoms. 
   To select all the atom indexes of the residues with some atoms within x :math:`\AA` from the ligand with residue
   index i (excluding the water molecules) the following command is used:

.. code-block:: none

 ((same resid as (protein within x of resid i)) or resid i) and not water

4) Use Save to save a list of atoms (and their residues) in a pdb file. Note that
   vmd does not preserve the atomic index from the original pdb file (which is the required
   ID needed for setting up the QM atoms). The correct atom indexes can be pulled directly 
   from the pdb file using the residue ID.

5) Use ``awk`` to get the atoms in the selected residues. e.g.

.. code-block:: none

 for res in 12 34 76 77
 do
    awk '$5 == $res' system.pdb >> QMatoms.pdb
    awk '$5~/$res/{print}' system.pdb >> QMatoms.pdb
 done

6) Use the get_qm_kind.py script (https://github.com/bioexcel/CP2K_qmmm_input_preparation_scripts/blob/main/get_qm_kind.py)
to convert the pdb containing the QM atoms to the format required in the CP2K input (requires python3).

.. code-block:: none

 python3 get_qm_kind.py QMatoms.pdb

Including water molecules
-------------------------

Generally, including water molecules in the QM region and treat them at quantum level
may be complicated. In fact, the water molecules usually move around more
readily than larger protein molecules and they may not remain far enough from the
boundaries of the QM box during the simulation, thus producing artefacts.
However in some circumstances water molecules may be important for the reaction chemistry
and/or can remain localised during the entire simulation (e.g. when coordinating a metal ion).
In this case negihbouring water molecules can be included in the selection using:

.. code-block:: none

 (same resid as (protein within x of resid i)) or resid i


---------------------------
Dealing with breaking bonds
---------------------------

When the boundary between the QM and MM region breaks a covalent bond, 
this bond has to be treated with care. CP2K offers some alternatives.
One of the most simple is adding a link atom to saturate the valence of the QM dangling atom.  
More about this can be found in the QMMM input part of the guide.
To avoid spurious polarisations at the boundary between the QM and MM part,
the bonds to be cut should not be chosen arbitrarily. When possible, one should 
break only C-C bonds, due to their symmetry and the total absence of electronegativity.
 
The bonds can be identified via visual inspection, e.g. with vmd or another pdb viewer, or by observation
of the pdb file. You should first find the residues within the QM region which are bonded
to residues treated without QM. Within these residues you can then find 
the C-C bond to break, and record the atomic indexes of the QM and MM C atoms.
These will be needed to correctly treat a QM-MM bond in CP2K (see section).


-------------------------------------
Expanding or shrinking the QM region
-------------------------------------

You may want to examine the effect of growing or shrinking the QM region on your
property of interest in order to decide on a suitable region size. If the calculation
is taking too long you could consider reducing the number of QM atoms in the region (i.e.
shrinking the region), or if the chemistry is not sufficiently included the region can be expanded.
This can be done by increasing or decreasing the  distance around the ligand (or region
of interest) using the above approach. The property of interest can be measured for different
QM region sizes and used to determine the optimum size. 

.. This approach has been documented in:

.. references





