==============================
 System preparation 
==============================

In this guide, we are going to briefly explain how to prepare your biological systems starting from a PDB file in the Protein Data Bank to obtaining topology and coordinates files ready to use in CP2K. 

--------------------------------------
Getting to know your biological system
--------------------------------------

Biological molecules are complex systems as they have evolved to react to changes in their chemical envinronment such as pH, concentration of substrates or voltage across biological membranes. All the factors that can tune the biological activity of interest must be taken into account for molecular modelling purposes. Therefore, a thourough literature search before starting your simulation will help you figure out which is the chemical environment of your biological system. 

In particular, QM/MM methods are often used to study chemical reactions catalysed by enzymes, light absorption through chromophores within proteins, reduction and/or oxidation of cofactors and much more. To study the aforementioned biological processes, it is crucial to known the fine details involved such as the protonation states and the geometry of the residues involved in the reaction, the presence of water molecules, the oxidation state of the chromophore or cofactor, coordination geometry of the metal ions involved... This structural information will also help to determine:

- The reaction coordinate that defines the chemical process of interest. 
- The starting configuration of your biological system.
- Which DFT method to use. 
- How big the QM region must be.


-----------------------
Preparing your PDB file
-----------------------

The most common way to obtain the structure of a biomolecule (proteins, nucleic acids, protein complexes...) is  via a structural biologist collaborator or via downloading an entry of the Protein Data Bank (PDB). Usually these structures are not ready to use as they will likely have crystallisation products and will require fixing to obtain the desired protonation state of the protein. 


Cleaning the structural coordinates of the biomolecule of interest
------------------------------------------------------------------

The structures should be cleaned of all those molecules that are not relevant to your study.  
It is worth mentioning here that crystallographic waters are very important for protein stability and enzymatic reactivity and therefore, they should not be discarded without a proper visual assessment. 

Asessing the protonation states of your biomolecule
---------------------------------------------------

As mentioned before, the protonation state of your system should be considered carefully. If you know the pH you want to simulate, you can get a suggestive protonation state for your protein using the pKa predictor. The results of these pKa predictors always have to be visually checked, as the protonation patterns are very important in enzymatic reactivity. 

Here it is a short list of pKa predictors:

- `PROPKA 3 <https://github.com/jensengroup/propka>`_: It predicts the pKa values of ionizable groups in proteins and protein-ligand complexes based on the 3D structure.
- `H++ server <http://biophysics.cs.vt.edu>`_: It computes pK values of ionizable groups in macromolecules and adds missing hydrogen atoms according to the specified pH of the environment.
- `DelphiPKa Web server <http://compbio.clemson.edu/pka_webserver/>`_: It allows to predict pKa's for ionizable groups in proteins, RNA and DNA.


Checking for disulphide bonds and other posttranslational modifications 
------------------------------------------------------------------------

Disulphide bonds keep some proteins properly folded, therefore it is very important to look for them in your protein structures. The cysteine residues involved in a disulphide bond should be rename to the aminoacid code CYX. 

It is also important to check for other posttranslational modifications such as phosporilation or glicosilation of protein residues. You might need to include them in your setup if they are relevant to your hypothesis. 


Presence of chromophores, cofactors, substrates, inhibitors and other organic molecules
---------------------------------------------------------------------------------------

Classical forcefields contain parameters for the most common residues in biomolecular simulations such as aminoacids, nucleic acids, lipids, sugars, ions and water molecules. Unfortunately, drug-like molecules, most chromophores and other organic molecules of interest of your biological system are not included in those forcefields and must be parameterised adhoc. 

There are several protocols to parameterise organic molecules for each forcefield as well as several web servers that provide parameters for organic molecules. We are going to list here a set of tutorials for ligand parameterisation and a list of web servers created for that same purpose. Additionally, we also include some databases of parameterised molecules. 

**Ligand parameterisation protocols:**

- `AMBER tutorials for developing non standard parameters <https://ambermd.org/tutorials/ForceField.php>`_: The AMBER project provides you with 6 tutorials on how to parameterise different non standard residues such as organic molecules, covalent ligands and metal ions. 
- CHARMM tutorials:
	- `All CHARMM tutorials <https://www.charmm.org/charmm/documentation/tutorials/>`_
	- `CHARMM tutorial on Parameterising novel residues <https://www.ks.uiuc.edu/Training/Tutorials/science/forcefield-tutorial/forcefield.pdf>`_
	- `Paratool VMD Plugin <http://www.ks.uiuc.edu/Research/vmd/plugins/paratool/>`_

**Web Servers:**

- `SwissParam <www.swissparam.ch>`_: a web server that provides topology and parameters for small organic molecules compatible with the CHARMM all atoms force field, for use with CHARMM and GROMACS.
- `CHARMM GUI Ligand Reader and Modeller <http://www.charmm-gui.org/?doc=input/ligandrm>`_: a web server that provides parameters compatible for CHARMM forcefields. 
- `ACPYPE Server <https://alanwilter.github.io/acpype/>`_ : a web server that generates topology parameters files for unusual organic chemical compounds for AMBER forcefields. 
- `LigParGen web Server <http://zarbi.chem.yale.edu/ligpargen/>`_ : is a web-based service that provides force field (FF) parameters for organic molecules or ligands with the OPLS-AA forcefield. 

**Published parameters of organic molecules:**

- `AMBER parameter database <http://research.bmh.manchester.ac.uk/bryce/amber/>`_ : Parameters for use  with the AMBER forcefield for an extensive list of cofactors and other organic molecules. 
- `AMBER DYES forcefield <https://github.com/t-/amber-dyes>`_ : Parameters for use with the AMBER forcefield for the most common dyes. 

Additionally, it is worth mentioning the `Open Force Field Initiative <https://openforcefield.org>`_ which is actively working to develop new forcefileds that escape from typical forcefield atomtypes and use chemical perception to parameterise organic molecules. 

------------------------------------------------
Preparing topology and coordinate files for CP2K
------------------------------------------------

CP2K allows several formats for topology files (you can find the complete list here: `&TOPOLOGY 
<https://manual.cp2k.org/trunk/CP2K_INPUT/FORCE_EVAL/SUBSYS/TOPOLOGY.html>`_ under the **&CONN_FILE_FORMAT** and the **&COORD_FILE_FORMAT** subsections). For biomolecular modelling purposes, the most convenient formats are AMBER formats (AMBER7 topology files, AMBER7 CRD files) and CHARMM formats (PSF, PDB). 

Since both AMBER and CHARMM software packages have excellent training material, here we are going to give a quick overview of the system preparation process and provide a list of useful tutorials for each software packages. We will highlight how to adapt those protocols to the specific requirements of CP2K:

- To use a cubic or triclinic box. 
- To neutralise the system. 
- To add missing forcefield parameters. 


System preparation using AMBERTools software package
----------------------------------------------------

`AMBERTools <https://ambermd.org/AmberTools.php>`_ is a free suite provided by the AMBER software package developers that provides all the tools needed to prepare a biological system. It includes AMBER forcefields for proteins, lipids, sugars, nucleic acids and drug-like molecules. Also provides all the tools needed to derive ad-hoc parameters for special residues such as chromophores and other organic molecules. AMBER also provides a lot of useful `tutorials <https://ambermd.org/tutorials/>`_. 

To showcase the process, we are going to provide an overview of the AMBERTools system preparation process as well as we encourage you to have a look to the `AMBER tutorials <https://ambermd.org/tutorials/>`_.

AMBERtools provides a tool named **LEap**, which is able to read coordinate files (such as PDB, MOL2, ...) and build AMBER topology and coordinate files (PARM7, RST7, ...). LEap comes in two flavours: **xleap** with a rudimentary GUI and **tleap** with only a terminal command line. You can provide commands either directly into the command line or as a list in a input file. 

A usual LEap input file will contain the following structure:

.. code-block:: none

  # Loading forcefield parameters
  #==================================
  # FF for Water and Counterions
  source leaprc.water.tip3p
  # FF for Proteins
  source leaprc.ff19SB
  # FF for Lipids (optional)
  source leaprc.lipid17
  # FF for organic molecules
  source leaprc.gaff 
  
  # Loading parameters for other ligands
  #==================================
  loadamberprep ligand.prepc
  ligand = loadMOL2 ligand.mol2 
  
  # Loading PDB coordinates
  #==================================
  # Load aligned coordinates of each part of the system
  prot = loadPDB protein.pdb 
  lig = loadPDB ligand.pdb
  waters = loadPDB xray_waters.pdb
  # Creating disulphide bonds
  bond NEW.80.SG   NEW.159.SG
  bond NEW.231.SG  NEW.235.SG
  # Combine all the coordin ates
  system = combine { prot lig waters }
  
  # Solvate and add counterions
  #==================================
  # Add a periodic box boundary and fill it with waters
  solvateBox system TIP3PBOX 12 iso
  # Neutralise
  addions2 system Cl- 0
  addions2 system Na+ 0 
  
  # Save AMBER input files
  #==================================
  savePDB system system.pdb
  saveAmberParm system system.parm7 system.rst7
  
  quit

You can execute the commands by using the following commands:

.. code-block:: bash

  tleap -f input.leap

For a complete list of LEap commands, please check the `AMBER documentation <https://ambermd.org/doc12/Amber20.pdf>`_ or the `LEap tutorial <http://ambermd.org/tutorials/pengfei/index.htm>`_ .


System preparation using CHARMM software package
------------------------------------------------
