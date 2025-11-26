Generating a Full Library of Origin Showers
===========================================

In ``generate_origin_library``, we provide the ``OriginLibraryGenerator`` class to
generate input steering files (INP, LIST, REAS) for a grid of zenith angles and
:math:`X_\mathrm{max}` values, which can then be simulated with CORSIKA/CoREAS.

In the following sections, we describe on how to run your own origin shower library.

How the library generator works
--------------------------------
The library generator creates steering files for a grid of zenith angles and
:math:`X_\mathrm{max}` values. While it is trivial to set the zenith angles, setting the :math:`X_\mathrm{max}` values
cannot be done directly in CORSIKA. Instead, we set the height of first interaction
to achieve the desired :math:`X_\mathrm{max}`. This is done using fitting functions
based on a large set of CONEX simulation, which (in general) depends on the site and 
hadronic interaction model used. The fitting functions are 
stored in the `first_height_parameters` directory.

Initializing the library generator
----------------------------------

The library generator depends on a few key parameters that are globally set for all
origin showers:
- `sim_primary` : the primary mass composition (e.g. "proton", "iron", etc.) of the cosmic ray initiating the shower.
- `sim_energy` : the primary energy of the cosmic ray initiating the shower (in GeV).
- `site` : the site configuration for the simulation (magnetic field and observation level). Currently, "lofar" and "auger" is supported.
- `atm_model` : the atmosphere model used for the simulation. Currently, only model numbers following those used in CORSIKA is supported, and GDAS atmospheres are not yet supported

The primary energy and mass composition are globally set for all simulations, as the template synthesis
method does not depend strongly on these parameters (due to air-shower universality). The site and atmosphere
settings are also fixed, as they define the magnetic field and observation level for the simulations.

In the future, we plan to add more site configurations.

Generating the steering files
-----------------------------

Once the library generator is initialized, we can set up the steering files for the generation of
the origin shower library.

Setting up paths
~~~~~~~~~~~~~~~~
First, we need to set up the paths where the steering files will be written to. This is done
using the `setup_paths` function. The function requires the parent path to the CORSIKA run executable,
the path where the steering files will be written to, the type of cluster in which the simulations
will be run, and the type of the hadronic interaction model used.

The cluster type is here so that the appropriate submission scripts can be generated. Currently, only "horeka" is supported.

To use this functionality, add your default submission script in the ``corsika/submit_templates``
directory, with the name ``submit_{cluster_type}.sh``.

The following arguments must be available in the submission script, in the given order:
1. Xmax
2. Zenith angle
3. the run directory
4. number of nodes to run the simulation on
5. number of cores to use per node
6. the name of the corsika executable
7. the run ID of the simulation 

Please take a look at the existing submission script for reference.

The hadronic interaction model is set to use the correct CORSIKA executable, and to import the correct fitting functions
used to generate the height of first interaction, which sets the :math:`X_\mathrm{max}` of the shower. Currently only 
"sibyll" (SIBYLL2.3d) and "qgsjetII" (QGSJETII-04) are supported.

.. warning::
    Currently, the following CORSIKA executable is used to generate the showers:
    ``mpi_corsika77550Linux_SIBYLL_urqmd_thin_coreas_parallel_runner``, and is hardcoded in the
    script.

    This executable can be compiled by including the MPI option with CORSIKA 7.7500, SIBYLL 2.3d,
    and UrQMD 1.3. Please refer to the CORSIKA documentation for more information on how to
    compile CORSIKA with these options.

Generating the steering files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After setting up the paths, we can generate the steering files for the full library of origin showers.
This is done using the `generate_submit_files` function. The function can set:

- a grid of zenith angles (minimum, maximum + step, step), which will generate linearly spaced zenith angles in degrees,
- a grid of :math:`X_\mathrm{max}` values (minimum, maximum + step, step), which will generate linearly spaced :math:`X_\mathrm{max}` values in g/cm\ :sup:`2`,
- the number of different random seeds to simulate for each configuration, to account for shower-to-shower fluctuations
- whether to generate the library only on the vxvxB arm (default) or on a rotated star-shape antenna layout.

Additionally, the number of nodes and number of cores per node should be set here.

Running this function will generate the following directory structure for a single library:
.. code-block:: text

    /path/to/steering_files/
    ├── sim_labels.txt
    ├── coreas_to_hdf5_origin.py
    ├── hdf5_sims/
    ├── zen0/
    │   ├── xmax600/
    |   |   ├── logs/
    │   │   ├── submit.sh
    │   │   ├── SIM000000/
    │   │   │   ├── INP file
    │   │   │   ├── LIST file
    │   │   │   └── REAS file
    │   │   ├── SIM000010/
    │   │   └── ...
    │   ├── xmax700/
    │   └── ...
    ├── zen3/
    └── ...

The generated files and directories are as follows:

- ``sim_labels.txt`` : a summary file containing the mapping between simulation number, zenith angle, :math:`X_\mathrm{max}`, and iteration number (random seed)
- ``coreas_to_hdf5_origin.py`` : a convenience script to convert the generated CoREAS output files to HDF5 format for use with template synthesis. This script is based on the ``coreas_to_hdf5.py`` script available in CoREAS, but modified to work with origin showers, where the samples are bandwidth limited to [30, 500] MHz and downsampled to 2 GHz. 
- ``hdf5_sims/`` : a directory where the converted HDF5 files will be stored
- ``zen{angle}/xmax{value}/SIM{sim_number}/`` : directories containing the steering files for each simulation, organized by zenith angle and :math:`X_\mathrm{max}` value.

The function returns the full set of paths to the generated steering files, which can be used to submit the simulations to the cluster.

Submitting the steering files
-----------------------------
Once the steering files are generated, you can submit the simulations to your computing cluster.

This can be conveniently done with the `submit_jobs` function, which takes the paths
to the steering files as input, and submits them to the cluster using the generated submission scripts.


Generating a single origin shower with the library
--------------------------------------------------
In addition to generating a full library of origin showers, the library generator can also be used to generate
a single origin shower with the `generate_single_origin_shower` function. This function takes the same
parameters as the full library generator, but only generates the steering files for a single
origin shower with the specified zenith angle, :math:`X_\mathrm{max}`, and simulation run number.

Example
=======
The following example shows how to generate a full library of origin showers with
zenith angles from 0 to 60 degrees in steps of 3 degrees, :math:`X_\mathrm{max}` values from 600 to 1200 g/cm\ :sup:`2` in steps of 100 g/cm\ :sup:`2`, and 3 different random seeds for each configuration, for antennas on the vxvxB axis.

.. code-block:: python

    from template_synthesis.corsika.generate_origin_library import OriginLibraryGenerator

    # Initialize the library generator
    library_generator = OriginLibraryGenerator(
        sim_primary="proton",
        sim_energy=1e8,  # in GeV
        site="lofar",
        atm_model=17 # US standard by Keilhauer
    )

    # Setup paths
    library_generator.setup_paths(
        corsika_runner_path="/path/to/corsika/executable",
        run_dir="/path/to/steering/files",
        cluster_type="horeka",  
        hadr_model="sibyll"
    )

    # Generate steering files for the full library
    steering_file_paths = library_generator.generate_submit_files(
        zenith_angles = (0, 63, 3),
        xmax_values = (600, 1300, 100),
        n_sims=3,
        n_nodes=2,
        n_cores_per_node=16,
        vxvxB = True
    )

    # Submit the jobs to the cluster
    library_generator.submit_jobs(steering_file_paths)