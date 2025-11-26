SMIETRun
========

The `SMIETRun` class is the simplest way to use template synthesis. It handles all necessary steps, including loading origin showers, loading target showers, synthesising them, and extracting relevant quantities.

Requirements
------------

To use this class, you must have the template library installed or have your own template library generated. If you have not done so yet, please refer to :doc:`ts_origin_showers` for guidance on how to obtain or generate the origin shower library.

The template library for the base version can be downloaded from `here <https://zenodo.org/record/15194465>`_ (approximately 13 GB when unpacked).

Initializing the SMIETRun Object
--------------------------------

The `SMIETRun` class can be initialized by giving a site name and backend. The site name should correspond to the site for which the origin showers were simulated (e.g., "base" for the base template library). The backend can be either "numpy" or "jax", depending on whether you want to use the NumPy or JAX implementation of the template synthesis algorithm.

.. code-block:: python

    from smiet import SMIETRun

    smiet_runner = SMIETRun(
        site="base",  # site name
        backend="numpy"  # backend: "numpy" or "jax"
    )

The sites currently supported are:

- "base": The base configuration (observation level at sea level, LOFAR magnetic field, U.S. standard atmosphere by B. Keilhauer with frequency range from [30, 500] MHz). 

.. note::
    The JAX backend is currently not supported here. Please fall back to the low-level implementation for the JAX version. 

(Optional) Generating Templates
-------------------------------

If you do not have the template library, but generated your own origin shower library, you can generate your own template library with this class. Only the path to the origin shower library and the desired place to save the template library is needed. This:

- loops over all origin showers in the origin shower library, indexed by their Xmax and zenith angle,

- generates the templates for each origin shower,

- saves the generated templates to the specified location with path "site/backend/xmax_zenith/template_simX.hdf5", where X is the number of the origin shower for that Xmax and zenith angle.

- Finally, it sorts the templates such that 'template_sim1.hdf5' will be the origin shower with the closest Xmax to the mean Xmax of all origin showers for that zenith angle. 

.. warning::
    Generating the templates can take a long time (at least 1.5 hours), depending on the number of origin showers available. 


.. code-block:: python

    path_to_origin_showers = '/path/to/origin/shower/library'
    path_to_save_templates = '/path/to/save/template/library'

    smiet_runner.generate_templates(
        origin_repo = path_to_origin_showers,
        template_repo = path_to_save_templates
    )

Loading the Templates
---------------------

With a given path to the template library, you can load the templates within a given Xmax and zenith angle range. Alternatively, you can also load templates from all available Xmax and zenith angles.

As each configuration (Xmax and zenith angle combination) can have multiple origin showers, you can choose to randomize the selection of origin showers for each configuration. If randomization is set to False, the first origin shower for each configuration (the closest one to the true Xmax) will be used.

.. code-block:: python

    path_to_templates = "/path/to/template/library"

    smiet_runner.load_templates(
        template_repo = path_to_templates,
        xmax_range = (700, 1000),  # in g/cm^2
        zenith_range = (36, 40),   # in degrees
        randomize = False          # whether to randomize the selection of origin showers
    )

Generating and Setting Target Showers
-------------------------------------

The target showers can be set in two ways:
1. Loading target showers from Coreas HDF5 files. You can provide a list of paths to Coreas HDF5 files, and the target showers will be loaded from these files (through smiet.CoreasShower).

.. code-block:: python

    smiet_runner.load_coreas_showers(
        coreas_shower_paths = [
            "/path/to/coreas/sim/file1.hdf5",
            "/path/to/coreas/sim/file2.hdf5",
            # add more paths as needed
        ]
    )

2. Manually setting the target showers by providing arrays of longitudinal profiles and zenith angles for each target shower. The grid of the atmospheric depth used to generate the longitudinal profiles must also be given. The grid need not match that of the origin shower, as internally the longitudinal profiles will be interpolated to the grid of the origin showers.

.. code-block:: python

    import numpy as np

    target_xmaxs = np.array([710.0, 800])
    target_zeniths = np.deg2rad(np.array([37.5, 38.5]))

    grammage_grid = np.linspace(0, 1100, 200)
    long_profs = []
    for xmax in target_xmaxs:
        long_profs.append( gaisser_hillas_function(grammage_grid, nmax = 1e8, x0 = 0.0, xmax=xmax, lmbda = 70.0))

    smiet_runner.generate_target_showers(
        target_grammage = grammage_grid,
        target_longs = long_profs,
        target_zeniths = target_zeniths
    )

Synthesizing the Target Showers
-------------------------------
Once the templates and target showers are set, you can synthesise the target showers. The synthesis will automatically find the Xmax and zenith angle configurations from the loaded templates that are closest (smaller) for each target shower.

If interpolation is set to True, the synthesis will be done with linear interpolation in Xmax. A separate synthesis will be done for each of the two bracketing Xmax configurations, and the final synthesized trace will be obtained by linear interpolation between these two traces, given by:

    trace_final = trace_lower + (trace_upper - trace_lower) * (Xmax_target - Xmax_lower) / (Xmax_upper - Xmax_lower)


If slices is set to True, the traces at each atmospheric slice will be saved instead of the trace at ground. This may be useful for analyses that want to explore the emission at different atmospheric depths.

Extracting Quantities
---------------------
After synthesizing the target showers, you can extract various quantities from the synthesized traces.

1. Synthesized traces

The traces can be extracted for given antenna positions (or all antenna positions) for a given target shower (specified by its Xmax and zenith angle).

A single antenna position can be extracted by providing the antenna name from .. code:`smiet_runner.antenna_labels`. All antenna positions can be extracted by setting ant_name to None.

If geoce is set to False, then the vxB / vx(vxB) polarization will be returned. This is computed through :doc:`ts_treatment_of_vxB`. If geoce is set to True, then the geomagnetic and charge-excess polarizations will be returned.

If slices is set to True during synthesis, the traces at each atmospheric slice can be extracted by providing the value of atmospheric depth to extract. Setting this to None (the default behaviour) will return the trace at ground level.

.. code-block:: python

    ant_name = "ant_r2_phi270"
    target_xmax_to_get, target_zeniths_to_get = smiet_runner.target_xmaxs[1], smiet_runner.target_zeniths[1]

    traces_geoce, times, _ = smiet_runner.get_traces(
        xmax=target_xmax_to_get, 
        zenith=target_zeniths_to_get, 
        ant_name=ant_name, 
        geoce=True, 
        slice_grammage=500
    )

2. Spectrum

The frequency spectrum can also be extracted in a similar manner, but returns the spectrum and the frequency grid instead.

.. code-block:: python

    spect_geoce, freq = smiet_runner.get_spectrum(
        xmax=target_xmax_to_get,
        zenith=target_zeniths_to_get,
        ant_name=ant_name,
        geoce=False,
        slice_grammage=None,
    )

3. Energy Fluence

The fluence can also be extracted easily for a given target shower. As before:

- if geoce is set to False, then the vxB / vx(vxB) polarization will be returned.
- if slices is set to True during synthesis, the fluence at each atmospheric slice can be extracted by providing the value of atmospheric depth to extract.

The function returns the fluences (in eV/m^2) at all antenna positions and the corresponding antenna positions in the shower plane.

.. code-block:: python

    fluences, ant_pos = smiet_runner.get_fluence(
        xmax=target_xmax_to_get,
        zenith=target_zeniths_to_get,
        geoce=False,
        slice_grammage=None
    )

Minimal Example
---------------

Here is a minimal example of how to use the `SMIETRun` class:

.. code-block:: python

    from smiet import SMIETRun, units
    import numpy as np

    import matplotlib.pyplot as plt  # for plotting routines
    
    # Define paths to origin templates and target shower data
    path_to_templates = "/path/to/template/library"

    # initialize the SMIETRun object
    smiet_runner = SMIETRun(
        site="base", backend="numpy"
    )

    # load templates within a given Xmax and zenith range
    smiet_runner.load_templates(
        template_repo = path_to_templates,
        xmax_range = (700, 1000),
        zenith_range = (36, 40),
        randomize = False
    )

    # load target showers from a coreas file
    smiet_runner.load_coreas_showers(
        coreas_shower_paths = [
            "/path/to/coreas/sim/hdf5/file"
        ]
    )

    # synthesize the traces
    smiet_runner.synthesize_showers(slices=False, interpolation=True)

    # get the traces for a given target and antenna position
    ant_name = "ant_r2_phi270"
    target_xmax_to_get, target_zeniths_to_get = smiet_runner.target_xmaxs[1], smiet_runner.target_zeniths[1]

    traces_geoce, times, _ = smiet_runner.get_traces(xmax=target_xmax_to_get, zenith=target_zeniths_to_get, ant_name=ant_name, geoce=True)

    # get the spectrum
    spect_geoce, freq = smiet_runner.get_spectrum(xmax=target_xmax_to_get, zenith=target_zeniths_to_get, ant_name=ant_name, geoce=True)

    # get the fluence
    fluences, ant_pos = smiet_runner.get_fluence(xmax=target_xmax_to_get, zenith=target_zeniths_to_get, geoce=False)

    fig, ax = plt.subplots(1,1, figsize=(5,4))
    sc = ax.scatter(
        ant_pos[:, 0], ant_pos[:, 1],
        c=fluences,
        cmap='inferno',
        edgecolor='white',
        alpha=1.0,
        vmin=0,
        vmax=45  # modify as needed
    )

    ax.set_xlabel("vxB / m", fontsize=14)
    ax.set_ylabel("vx(vxB) / m", fontsize=14)

    ax.set_xlim(-200, 200)
    ax.set_ylim(-200, 200)

    ax.tick_params(axis='both', which='major', labelsize=12)

    cbar = fig.colorbar(sc, ax=ax)
    cbar.ax.set_ylabel("Fluence / eV m$^{-2}$", fontsize=16)
    cbar.ax.tick_params(labelsize=12)


API documentation
-----------------

.. toctree::
   :maxdepth: 1

   ../apidoc/smiet.core