==========================
Selecting QM atoms
==========================



-----------------
Initial selection
-----------------

The starting point for deciding which atoms should be treated with QM is the ligand, or
region of interest in you system (such as an metal ion). The size of the ligend will usually 
dictate whether you can treat the whole ligend, a subsection of the ligend, 
or the ligend plus some neighbouring residues with QM. As previously mentioned, ideally the 
number of QM atoms should not be too large (over 200 QM atoms) as this will become too 
computationally intensive. However we want to be able to include all regions 
involved in the chemical reaction as much as possible.

A good approach for selecting the QM region is therefore to select the entire ligand plus
some residues nearby, say within x Angstrom of the ligand. The size you consider for this region
around the ligand will depend on the number of atoms in your ligand. If you have a ligand of
50 or more atoms then you might only include residues within 2-3 Angstroms, in order to limit 
the overall size of the QM region. For a smaller ligand you may be able to include more atoms around it
(up to approximately 6 Angstoms).


Creating the QM atoms list for CP2K
-----------------------------------

Creating a list of  QM atoms that can be read by CP2K can be done using vmd. (LINK)

1) Open the entire system pdb file with ``vmd``.

.. code-block:: none

 vmd system.pdb

2) Go to File >> Save coordinates

3) Use Selected Atoms to save a pdb containing the desired QM atoms. 
To select atoms in entire residues within x Angstrom around a ligand with residue
index i (excluding water) the following command is used:

.. code-block:: none

 ((same resid as (protein within x of resid i)) or resid i) and not water

4) Use Save to save a list of atoms (and their residues) in a pdb file. Note that
vmd does not preserve the atomic index from the original pdb file (wihch is the required
ID needed for setting up the QM atoms). To get the correct the atoms can be pulled directly 
from the pdb file using the residue ID.

5) Use ``awk`` to get the atoms in the selected residues. e.g.

.. code-block:: none

 for res in 12 34 76 77
 do
    awk '$5 == $res' system.pdb >> QMatoms.pdb
    awk '$5~/$res/{print}' system.pdb >> QMatoms.pdb
 done

6) Use the get_qm_kind.py script (path/to/script) to convert the pdb containing the QM 
atoms to the format required in the CP2K input (requires python).

.. code-block:: none

 python get_qm_kind.py QMatoms.pdb

Including waters
----------------

Generally including water atoms in the QM region is more complicated as they move around 
more readily than proteins and may not stay within the QM cell (see section on this).
However in some circumstances water atoms may be important to the reaction chemistry.
In this case negihbouring water molecules can be included in the selection using:

.. code-block:: none

 (same resid as (protein within x of resid i)) or resid i


---------------------------
Dealing with breaking bonds
---------------------------

Across the boundary between QM and MM atoms the broken bonds have to be deal with 
correctly. In CP2K this involves adding link atoms where the dangling QM bond is capped. 
More about this can be found in the QMMM input part of the guide.
The boundary between the QM and MM part should not be arbitrarily chosen, as to avoid 
charge transfer between the QM and MM parts. Based on this breaking a C-C bond 
is usually a good choice.
 
The bonds can be identied through visualisation, e.g. with vmd or other pdb viewer, or by observation
of the pdb file. You should first find the residues within the QM region which are bonded
to residues which are treated without QM. Within these residues you can then find 
the C-C bond to break, and record the atomic indexes of the QM and MM C atoms.
These will be needed to correctly treat a QM-MM bond in CP2K (see section).


----------------------------------
Expanding or shrinking the QM region
----------------------------------

You may want to examine the effect of growing or shrinking the QM region on your
property of interest in order to decide on a suitable region size. If the calculation
is taking too long you could consider reducing the number of QM atoms in the region (i.e.
shrinking the region), or if the chemistry is not sufficiently included the region can be expanded.
This can be done by increasing or decreasing the  distance around the ligand (or region
of interest) using the above approach. The property of interest can be measured for different
QM region sizes and used to determine the optimum size. This approach has been 
documented in:

.. references





