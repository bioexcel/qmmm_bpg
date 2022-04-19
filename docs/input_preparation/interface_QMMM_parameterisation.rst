=============================
QM/MM parameterisation
=============================

This chapter describes how to set up a QM/MM system within the GROMACS/CP2K interface.
It assumes that you have a equilibriated GROMACS coordinates file (``.gro``), a GROMACS topolopy
file (``.top``) and a MD parameter file (``.mdp``) with settings to do a classical MM
simulation. From this this guide will show you how to select your QM atoms, 
and set the parameters in order to run a QM/MM simulation. 

----------------------
Selecting QM atoms
----------------------

The atoms for the QM region must be given through a GROMACS index file (``.ndx``). You can
select a group of atoms and then name the group appropirately e.g. QMatoms.


For example to select all non-water atoms to be treated with QM:

.. code-block:: none

  gmx_mpi make_ndx -f ${INPUT}.gro


  Reading structure file
  Going to read 0 old index file(s)
  Analysing residue names:
  There are:     1      Other residues
  There are:  2384      Water residues
  Analysing residues not classified as Protein/DNA/RNA/Water and splitting into groups...

  0 System              :  7186 atoms
  1 Other               :    34 atoms
  2 Protein             :    34 atoms
  3 Water               :  7152 atoms
  4 SOL                 :  7152 atoms
  5 non-Water           :    34 atoms


   nr : group      '!': not  'name' nr name   'splitch' nr    Enter: list groups
   'a': atom       '&': and  'del' nr         'splitres' nr   'l': list residues
   't': atom type  '|': or   'keep' nr        'splitat' nr    'h': help
   'r': residue              'res' nr         'chain' char
   "name": group             'case': case sensitive           'q': save and quit
   'ri': residue index

  > name 5 QMatoms


This group name should then match the name set in the ``qmmm-cp2k-qmgroup`` name
in the ``.mdp`` file. The index file will need to be provided in the grompp command.

------------------------------
Gromacs QM/MM input parameters
------------------------------

The main CP2K QMMM parameters for the QM/MM ``.mdp`` file are shown below:


.. code-block:: none

  ; CP2K QMMM parameters
  qmmm-cp2k-active              = true    ; Activate QMMM MdModule
  qmmm-cp2k-qmgroup             = QMatoms ; Index group of QM atoms
  qmmm-cp2k-qmmethod            = PBE     ; Method to use
  qmmm-cp2k-qmcharge            = 0       ; Charge of QM system
  qmmm-cp2k-qmmultiplicity      = 1       ; Multiplicity of QM system


qmmm-cp2k-active
----------------

Set to true to turn on QMMM.

qmmm-cp2k-qmgroup
-----------------

The group name for the QM atoms as in the index file.

qmmm-cp2k-qmmthod
-----------------

The method to use for the QM atoms. At present the available options are:

* PBE - Use the PBE method in CP2K.

* BLYP - Use the BLYP method in CP2K.

* INPUT - pass your own CP2K input file contain the QM set up. See below.

------------------
Using a CP2K input
------------------

If the functional method you need is not available or if you wish to change 
any of the QM settings then you should provide your own CP2K input file.

The best way to generate this is to first run the inferface with either the
PBE or BLYP option set to create a CP2K input file with the interface. You 
can then use this file as a template and edit it as you wish.

To use this CP2K input file rather than generate a new one you should set
the ``qmmethod`` to ``INPUT`` in the ``.mdp`` file as shown below:


.. code-block:: none

  qmmm-cp2k-qmmethod            = INPUT    ; use own cp2k.inp


To generate the tpr file for running you also need to add the name of the cp2k
input file with the ``-qmi`` option:

.. code-block:: none

  gmx grompp -f sys.mdp -p sys.top -c sys.gro -n sys.ndx -qmi sys_cp2k.inp -o sys.tpr

You may then run the ``.tpr`` as usual.
