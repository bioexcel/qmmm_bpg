==============================
 System preparation 
==============================

In this guide, we are going to briefly explain how to prepare your biological systems starting from a PDB file in the Protein Data Bank to obtaining topology and coordinates files ready to use in CP2K. 

------------
Getting to know your biological system
------------

Biological molecules are complex systems as they have evolved to react to changes in their chemical envinronment such as pH, concentration of substrates or voltage across biological membranes. All the factors that can tune the biological activity of interest must be taken into account for molecular modelling purposes. Therefore, a thourough literature search before starting your simulation will help you figure out which is the chemical environment of your biological system. 

In particular, QM/MM methods are often used to study chemical reactions catalysed by enzymes, light absorption through chromophores within proteins, reduction and/or oxidation of cofactors and much more. To study the aforementioned biological processes, it is crucial to known the fine details involved such as the protonation states and the geometry of the residues involved in the reaction, the presence of water molecules, the oxidation state of the chromophore or cofactor, coordination geometry of the metal ions involved... This structural information will also help to determine:

- The reaction coordinate that defines the chemical process of interest. 
- The starting configuration of your biological system.
- Which DFT method to use. 
- How big the QM region must be.


------------
Preparing your PDB file
------------

The most common way to obtain the structure of a biomolecule (proteins, nucleic acids, protein complexes...) is  via a structural biologist collaborator or via downloading an entry of the Protein Data Bank (PDB). Usually these structures are not ready to use as they will likely have crystallisation products and will require fixing to obtain the desired protonation state of the protein. 

Cleaning the structural coordinates of the biomolecule of interest
------------

The structures should be cleaned of all those molecules that are not relevant to your study.  

Asessing the protonation states of your biomolecule
------------

As mentioned before, the protonation state of your system should be considered carefully. If you know the pH you want to simulate, you can get a suggestive protonation state for your protein using the pKa predictor. The results of these pKa predictors always have to be visually checked, as the protonation patterns are very important in enzymatic reactivity. 

Here it is a short list of pKa predictors:

- `PROPKA 3 <https://github.com/jensengroup/propka>`_: It predicts the pKa values of ionizable groups in proteins and protein-ligand complexes based on the 3D structure.
- `H++ server <http://biophysics.cs.vt.edu>`_: It computes pK values of ionizable groups in macromolecules and adds missing hydrogen atoms according to the specified pH of the environment.
- `DelphiPKa Web server <http://compbio.clemson.edu/pka_webserver/>`_: It allows to predict pKa's for ionizable groups in proteins, RNA and DNA.


Checking for disulphide bonds and other posttranslational modifications 
------------

Disulphide bonds keep some proteins properly folded, therefore it is very important to look for them in your protein structures. The cysteine residues involved in a disulphide bond should be rename to the aminoacid code CYX. 

It is also important to check for other posttranslational modifications such as phosporilation or glicosilation of protein residues. You might need to include them in your setup if they are relevant to your hypothesis. 


------------
Preparing topology and coordinate files for CP2K
------------

CP2K allows several formats for topology files (you can find the complete list here: `&TOPOLOGY 
<https://manual.cp2k.org/trunk/CP2K_INPUT/FORCE_EVAL/SUBSYS/TOPOLOGY.html>`_ under the **&CONN_FILE_FORMAT** and the **&COORD_FILE_FORMAT** subsections). For biomolecular modelling purposes, the most convenient formats are AMBER formats (AMBER7 topology files, AMBER7 CRD files) and CHARMM formats (PSF, PDB). 

Since both AMBER and CHARMM software packages have excellent training material, here we are going to give a quick overview of the system preparation process and provide a list of useful tutorials for each software packages. We will highlight how to adapt those protocols to the specific requirements of CP2K:

- To use a cubic or triclinic box. 
- To neutralise the system. 
- To add missing forcefield parameters. 


System preparation using AMBERTools software package
-------------

`AMBERTools <https://ambermd.org/AmberTools.php>`_ is a free suite provided by the AMBER software package developers that provides all the tools needed to prepare a biological system. It includes AMBER forcefields for proteins, lipids, sugars, nucleic acids and drug-like molecules. Also provides all the tools needed to derive ad-hoc parameters for special residues such as chromophores and organic products. AMBER also provides a lot of useful `tutorials <https://ambermd.org/tutorials/>`_. 





System preparation using CHARMM software package
----------------------