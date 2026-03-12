***************************
AL starting from TS
***************************

Active learning can be performed starting from a fixed initial configuration (e.g. transition state). 
To perform this, you must already have the TS structure (here cis_endo_TS_PBE0.xyz) in your working directory.

----------------
Training the model 
----------------
First, let's load the TS structure in xyz format:
.. code-block:: python
    import mlptrain as mlt

    system = mlt.System(
        mlt.Molecule('cis_endo_TS_PBE0.xyz', charge=0, mult=1), box=None
    )

Then we can define the MLIP model, specify its save name and train it via the AL loop:
.. code-block:: python
    mace = mlt.potentials.MACE('endo_ace', system=system)

    mace.al_train(
        method_name='xtb',      # electronic structure used
        temp=500,               # temperature in K used during the MLIP-MD run
        max_active_time=1000,   # condition upon which AL stops
        fix_init_config=True,   # ensures that each AL loop will start from the TS
        keep_al_trajs=True,     # saves all the trajectories sampled during each AL loop
    )

``mace.al_train`` creates a folder that will contain all the trajectories generated, saved as traj_iter{n}_{m}.xyz were n is the AL iteration and m is the index of the trajectory.


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
    trajectory.save(filename='DA_CP-MVK_trajectory.xyz')

    # Check the parity plots 
    trajectory.compare(mace, 'xtb')

