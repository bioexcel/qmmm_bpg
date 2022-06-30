==============================
 System preparation 
==============================

In this guide, we are going to briefly explain how to prepare your biological systems starting from a PDB file in the Protein Data Bank to obtaining topology and coordinates files ready to use in CP2K.

You may also like to consult our guide on Best Practices in QM/MM Simulation of Biomolecular Systems, which gives advice on QM/MM methodology that is independent of the software used: https://docs.bioexcel.eu/qmmm_simulation_bpg


--------------------------------------
Getting to know your biological system
--------------------------------------

Biological molecules are complex systems as they have evolved to react to changes in their chemical environment such as pH, concentration of substrates or voltage across biological membranes. All the factors that can tune the biological activity of interest must be taken into account for molecular modelling purposes. Therefore, a thorough literature search before starting your simulation will help you figure out which is the chemical environment of your biological system. 

In particular, QM/MM methods are often used to study chemical reactions catalysed by enzymes, light absorption through chromophores within proteins, reduction and/or oxidation of cofactors and much more. To study the aforementioned biological processes, it is crucial to known the fine details involved such as the protonation states and the geometry of the residues involved in the reaction, the presence of water molecules, the oxidation state of the chromophore or cofactor, coordination geometry of the metal ions involved... This structural information will also help to determine:

- The reaction coordinate that defines the chemical process of interest. 
- The starting configuration of your biological system.
- Which DFT method to use. 
- How big the QM region must be.


-----------------------
Preparing your PDB file
-----------------------

The most common way to obtain the structure of a biomolecule (proteins, nucleic acids, protein complexes...) is  via a structural biologist collaborator or by downloading an entry from the Protein Data Bank (PDB). Usually these structures are not ready to use as they will likely have crystallisation products and will require fixing to obtain the desired protonation state of the protein. 


Cleaning the structural coordinates of the biomolecule of interest
------------------------------------------------------------------

The structures should be cleaned of all those molecules that are not relevant to your study.  
It is worth mentioning here that crystallographic waters are very important for protein stability and enzymatic reactivity and therefore, they should not be discarded without a proper visual assessment. 

Assessing the protonation states of your biomolecule
----------------------------------------------------

As mentioned before, the protonation state of your system should be considered carefully. If you know the pH you want to simulate, you can get a suggested protonation state for your protein using the pKa predictor. The results of these pKa predictors always have to be visually checked, as the protonation patterns are very important in enzymatic reactivity. 

Here it is a short list of pKa predictors:

- `PROPKA 3 <https://github.com/jensengroup/propka>`_: It predicts the pKa values of ionizable groups in proteins and protein-ligand complexes based on the 3D structure.
- `H++ server <http://biophysics.cs.vt.edu>`_: It computes pK values of ionizable groups in macromolecules and adds missing hydrogen atoms according to the specified pH of the environment.
- `DelphiPKa Web server <http://compbio.clemson.edu/pka_webserver/>`_: It allows to predict pKa's for ionizable groups in proteins, RNA and DNA.


Checking for disulphide bonds and other posttranslational modifications 
------------------------------------------------------------------------

Disulphide bonds keep some proteins properly folded, therefore it is very important to look for them in your protein structures. The cysteine residues involved in a disulphide bond should be renamed to the aminoacid code **CYX**. It is worth mentioning that MM forcefields encode each chemical species of each aminoacid residue using a three-letter-code. Since cysteines involved in a disulphide bond lack the hydrogen atom attached to the S atom, they must be named differently (CYX) than the normal cysteine residues (CYS). 

It is also important to check for other posttranslational modifications such as phosphorilation or glicosilation of protein residues. You might need to include them in your setup if they are relevant to your hypothesis. 


Presence of chromophores, cofactors, substrates, inhibitors and other organic molecules
---------------------------------------------------------------------------------------

Classical forcefields contain parameters for the most common residues in biomolecular simulations such as aminoacids, nucleic acids, lipids, sugars, ions and water molecules. Unfortunately, drug-like molecules, most chromophores and other organic molecules of interest of your biological system are not included in those forcefields and must be parameterised explicitly. 

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

Additionally, it is worth mentioning the `Open Force Field Initiative <https://openforcefield.org>`_ which is actively working to develop new forcefields that escape from typical forcefield atomtypes and use chemical perception to parameterise organic molecules. 

The parameters for the small molecules obtained with the aforementioned methods are probably not as accurate as the rest of the MM forcefield used in the setup. There is a lot of effort in the parameterisation and validation of forcefields for proteins, nucleic acids and solvent molecules. However, if you plan to include that small molecule in the QM region in the final QM/MM simulation, it will probably be enough to successfully equilibrate the system at the MM level. In QM/MM simulation, these parameters will be no longer used as the molecule will be described using a more accurate QM level of theory. 

If that is not your case and your small molecule will be included in the MM region, a more extensive validation of the MM parameters is required. Usually running a short molecular dynamics simulation of the solvated ligand can expose problems in the parameterisation process.     


------------------------------------------------
Preparing topology and coordinate files for CP2K
------------------------------------------------

CP2K allows several formats for topology files (you can find the complete list here: `&TOPOLOGY 
<https://manual.cp2k.org/trunk/CP2K_INPUT/FORCE_EVAL/SUBSYS/TOPOLOGY.html>`_ under the **&CONN_FILE_FORMAT** and the **&COORD_FILE_FORMAT** subsections). For biomolecular modelling purposes, the most convenient formats are AMBER formats (AMBER7 topology files, AMBER7 CRD files) and CHARMM formats (PSF, PDB). 

Since both AMBER and CHARMM software packages have excellent training material, here we are going to give a quick overview of the system preparation process and provide a list of useful tutorials for each software package. We will highlight how to adapt those protocols to the specific requirements of CP2K.

**1) System building**

It is crucial to build your model system in a way that represents the biological process you want to study in the most accurate way possible. You should include all of the key elements of your system you have investigated beforehand. 

In order to prepare a suitable model system, you should:
- Include water molecules when possible
- Create bonds for special features as disulphide bonds, covalent molecules 
- Use a cubic or triclinic periodic box (This is a requirement to run QM/MM simulations in parallel in CP2K.)
- Neutralise the system in order to avoid simulation artefacts. 


**2) Minimisation, thermalisation and equilibration using MM forcefields**

After you have built your topology and coordinate files, you must minimise these coordinates using the forcefield parameters in your topology file. Energy minimisation will find an energy minimum in the potential energy surface of your system and fix any possible bad contacts in your initial structure. If possible, it is important that you use minimise using a steepest descent algorithm at first to avoid getting stuck in a local minimum, and subsequently refine the structure by minimising with the conjugate gradient algorithm.  

Once the system is minimised, it has to be subsequently heated (from 0K to your target conditions e.g. 300K) and equilibrated. Since a sudden increase in the kinetic energy of your system may lead to system instabilities, a gradual and slow heating process is recommended where possible. 

Afterwards the pressure and volume of the system must be equilibrated. However, the nature of your simulation (for instance globular and membrane proteins) might require a specific equilibration recipe. Therefore, we will point out at the end of this page several tutorials that cover the specifics of each kind of simulation. 

As a general rule, you should check that all the fixed quantities of the ensemble that you use (NVT, NPT, NVE ...) are stable before you start your production runs. It is also wise to assess the stability of your biomolecule during all the themalisation and equilibration process. 

It is worth mentioning that the equilibrated system using MM forcefield will be equilibrated at this level of theory only, it will have to be equilibrated again at the QM/MM level of theory (see **4) Monitorisation using QM/MM methods** ) before starting the QM/MM production runs.


**3) Adding missing parameters to the MM forcefield**

The current AMBER and CHARMM forcefields were developed to reproduce the behaviour of biomolecules using classical mechanics. In this context, hydrogen atoms of standard residues and water molecules are parameterised without Lennard-Jones parameters. MM forcefields account for these missing parameters and simulations are usually performed with hydrogen restraining algorithms such as SHAKE, SETTLE or LINKS that freeze the X-H bond vibration frequency in order to increase the simulation timestep. 

However, these approximations cannot be done in quantum mechanics. In particular, in QM/MM simulations they lead to unnatural interactions between the point charges of the MM subsystem with the electronic densities of the QM subsystem, which eventually cause the simulation to crash. Therefore, you have to add the missing parameters for hydrogens at least in the QM/MM interface. 

There are two ways to add the Lennard-Jones parameters to the forcefield:

- to add the Lennard-Jones parameters in the CP2K input file within the &QMMM subsection of the &FORCE_EVAL section. You must specify the Lennard-Jones parameters for each kind of pairwise interaction involving an hydrogen atom using the following format:

.. code-block:: none

  &QMMM
  ...
    &FORCEFIELD
      &NONBONDED
        &LENNARD-JONES
            ATOMS HW O
            EPSILON [kcalmol] 0.058
            SIGMA [angstrom]  2.2612
            RCUT [angstrom] 9.0
        &END
      &END
    &END

- to modify the Lennard-Jones parameters directly in the topology file. AMBERtools provides a tool to modify PARM7 topology files named **parmed**. More details on how to do this can be found in the `parmed documentation <https://parmed.github.io/ParmEd/html/index.html>`_ .

Also, if you are using a QM region that shares a covalent bond with the MM region, you must make sure that the MM subsystem remains neutral as a charge imbalance in the MM subsystem can lead to important simulation artefacts. Therefore you must modify the charges of the MM region, usually those of molecule that is split between the two regions. If you are using an AMBER topology, you can easily modify the topology using **parmed**.  



**4) Monitorisation using QM/MM methods**

Once the topology is amended and coordinates of the system are properly equilibrated, we are ready to start the QM/MM simulations. It is recommended to perform a short monitorisation simulation using the QM/MM of choice before starting the production runs in order to assess the stability of the QM/MM interface. 



System preparation using AMBERTools software package
----------------------------------------------------

`AMBERTools <https://ambermd.org/AmberTools.php>`_ is a free suite provided by the AMBER software package developers that provides all the tools needed to prepare a biological system. It includes AMBER forcefields for proteins, lipids, sugars, nucleic acids and drug-like molecules. Also provides all the tools needed to derive ad-hoc parameters for special residues such as chromophores and other organic molecules. AMBER also provides a lot of useful `tutorials <https://ambermd.org/tutorials/>`_. 

AMBER also provide detailed tutorials for different kinds of biomolecules:

- `Nucleic acids <https://amberhub.chpc.utah.edu/analisis-of-nucleic-acid-simulation/>`_
- `Globular proteins <http://ambermd.org/tutorials/basic/tutorial0/index.php>`_ 
- `Membrane proteins <https://ambermd.org/tutorials/advanced/tutorial16/index.php>`_ 

System preparation using CHARMM software package
------------------------------------------------

`CHARMM <https://www.charmm.org/charmm/>`_ (Chemistry at HARvard Molecular Mechanics) is a molecular simulation program developed with a primary focus on molecules of biological interest. CHARMM contains a comprehensive set of analysis and model building tools. CHARMM also has a lot of useful `tutorials <https://www.charmm.org/charmm/documentation/tutorials/>`_ .

CHARMM has several tutorials to perform MD simulations of biomolecules:

- `CHARMM GUI web server <http://www.charmm-gui.org/>`_
- `Globular proteins <https://www.charmmtutorial.org/index.php/Full_example>`_
- `Membrane proteins <http://www.charmm-gui.org/?doc=tutorial&project=membrane>`_
