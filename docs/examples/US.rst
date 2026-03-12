***************************
Umbrella Sampling
***************************

This example assumes that you already have an MLP model (e.g. endo_ace_stagetwo), the coordinates of the TS (e.g. cis_endo_TSE_pbe0.xyz) and a trajectory file from which to initialise the umbrella over (e.g. icr_IRC_Full_trj.xyz) in your working directory.

First, let's set some parameters for the Umbrella Sampling.
.. code-block:: python
    import mlptrain as mlt
    import numpy as np
    from mlptrain.box import Box
    from mlptrain.log import logger

    us = mlt.UmbrellaSampling(zeta_func=mlt.AverageDistance((1, 12), (6, 11)), kappa=10)
    temp = 300

Then, we load the trajectory, the TS structure and define the system:
.. code-block:: python
    irc = mlt.ConfigurationSet()
    irc.load_xyz(filename="irc_IRC_Full_trj.xyz", charge=0, mult=1)

    for config in irc:
        config.box = Box([100, 100, 100])

    irc.reverse()

    TS_mol = mlt.Molecule(name="cis_endo_TS_PBE0.xyz", charge=0, mult=1, box=None)

    system = mlt.System(TS_mol, box=Box([100, 100, 100]))

Let's load the already trained MLP model:
.. code-block:: python
    endo = mlt.potentials.MACE("endo_ace_stagetwo", system)

You can now run your US simulation:
.. code-block:: python
    us.run_umbrella_sampling(
        irc,
        mlp=endo,
        temp=temp,
        interval=5,
        dt=0.5,
        n_windows=15,
        init_ref=1.55,
        final_ref=4,
        ps=10,
    )
    us.save("wide_US")

    us_final = mlt.UmbrellaSampling.from_folders("wide_US", temp=temp)
    us_final.wham()


