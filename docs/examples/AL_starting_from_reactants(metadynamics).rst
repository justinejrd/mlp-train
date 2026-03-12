***************************
AL starting from reactants (metadynamics)
***************************

With this method, you can start the AL sampling from any point of the PES. 
This method requires an initial configuration for metadynamics (e.g. ch3cl_f.xyz) located in your working directory.

----------------
Training of the model
----------------
First, you need to specify the system and define the MLIP model you want to train and its save name:
.. code-block:: python
    import mlptrain as mlt

    system = mlt.System(mlt.Molecule("ch3cl_f.xyz", charge=-1, mult=1), box=None)

    mace = mlt.potentials.MACE("r1_wtmetad", system=system)

Then, we need to define the collective variable of the reaction that will be biased during the weel-tempered metadynamics (WTMetaD) process:
.. code-block:: python
    # Define Collective Variable for WTMetaD AL (r_cl - r_f)
    diff_r = mlt.PlumedDifferenceCV(name="diff_r", atom_groups=((0, 2), (0, 1)))

We also define a second auxiliary CV, that won't be biased during WTMetaD but to which we attach an upper-wall retraint to limit its range.
.. code-block:: python
    # Define Collective Variable and attach an upper wall
    avg_r = mlt.PlumedAverageCV(name="avg_r", atom_groups=((0, 1), (0, 2)))
    avg_r.attach_upper_wall(location=2.5, kappa=1000) # attached upper wall bias located at 2.5 A with a 1k eV/A² harmonic constraint

Let's generate the input files for Plumed:
.. code-block:: python
    # Initialise PlumedBias for WTMetaD AL
    bias = mlt.PlumedBias(cvs=(avg_r, diff_r))

Then we need to specify the CV biased during WTMetaD and parameters of the deposited Gaussian (width and bias factor):
    .. code-block:: python
    bias.initialise_for_metad_al(width=0.05, cvs=diff_r, biasfactor=100)

Finally, you can train your MACE model.
.. code-block:: python
    # Define the potential and train using WTMetaD AL 
    mace.al_train(
        method_name="xtb",
        temp=300,
        n_init_configs=5,
        n_configs_iter=5,
        max_active_iters=50,
        min_active_iters=30,
        inherit_metad_bias=True,    # ensures that bias from previous iterations is brought to the following iterations
        bias=bias,
    )

The data points obtained are saved in r1_wtmetad_al.xyz.


----------------
Validation of the model  
----------------
You can run a short NVT simulation to check the performance of the trained model:
.. code-block:: python
    # Run some dynamics with the potential
    trajectory = mlt.md.run_mlp_md(
        configuration=system.configuration,
        mlp=mace,
        fs=200,
        temp=300,
        dt=0.5,
        interval=10,
    )

    # Save the trajectory file
    trajectory.save(filename='CH3Cl_F_SN2_trajectory.xyz')
    
    # Check the parity plots 
    trajectory.compare(mace, 'xtb')