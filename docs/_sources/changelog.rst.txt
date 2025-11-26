Release notes
=============

Version 0.8.0
-------------

In this release, we introduce several new features and improvements to the SMIET package, enhancing its functionality and user experience. Additionally, most parity issues between the Numpy and JAX implementations have been resolved.

Highlights and breaking changes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- Added the SMIETRun class, which provides a high-level interface for running the entire template synthesis workflow, from loading origin and target showers to synthesizing traces and extracting relevant quantities.
- Resolved most parity issues between the Numpy and JAX implementations, ensuring consistent behavior and results across both backends.

Resolved parity issues
^^^^^^^^^^^^^^^^^^^^^^^
- Loading / saving templates : this is both done with HDF5 files, including compression for large arrays.
- Traces are now band-pass filtered at initialization (JAX) or at access level (numpy) to 30 - 500 MHz by default, as frequencies outside this range is not relevant with SMIET.
- antenna-specific information in synthesis are contained in antenna_information.
- The frequency array for TemplateSynthesis are loaded consistently (without units.MHz).
- Antenna properties from shower objects are stored as antenna_arrays.
- a function to interpolate the longitudinal profile to the atmospheric grid used in the origin shower has been added to both versions.
- naming conventions (slice_grammages to grammages, long to long_profile, magnetic_field_vector to magnetic_field).
- long_profile now returns only the longitudinal profile, and the grammages are set as a separate class object.
- the Gaisser-Hillas parameters are now set as a dictionary, which store all relevant parameters.
 
Version 0.7.0
-------------

In this release, we introduce the origin shower generation functionality, allowing users to create their own origin shower libraries for template synthesis. Several bug fixes and improvements have also been made to enhance the overall stability and performance of the package.

Highlights and breaking changes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- Added functionality to generate origin shower libraries
- Added functionality to generate CORSIKA input files for origin showers with given Xmax, based on fitting functions
- Added functionality to transform traces synthesised on the vxvxB axis to a star-shape pattern
- Bugfix in setting the time axis of the JAX version, which previous translated the time axes to the middle (for zero padding). This is now no longer done, which also preserves the timing between different showers.
- grammage units are now uniformly set to g/cm^2 throughout the code. i.e. no internal units are used for grammage.
- Improvements to the CORSIKA file generation functions.


Version 0.6.0
-------------

Highlights and breaking changes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- The Numpy version now has support for GDAS atmospheres.
- A new `smiet.numpy.io.CoreasShower` class was added, that provides a more convenient interface for
  working with CoREAS (not necessarily sliced) showers.
- Fixed a bug in the `smiet.numpy.geo_ce_to_e()` function, which resulted in an incorrect transformation.
- Fixed a bug in the JAX version where the azimuth angle was not set correctly, causing star-shaped antenna arrays
  to be off.
- Both versions now have a utility function called `transform_traces_on_vxB()` which can be used to get
  the traces on the vxB axis, where the template synthesis does not work.
- Improvements to the CORSIKA file generation functions.
- A demo folder has been added to the package, which contains example scripts to help users get started with
  the package. Together with, some origin showers have been made available for download.

Overview of all changes
^^^^^^^^^^^^^^^^^^^^^^^

Numpy module
~~~~~~~~~~~~

1. A new `smiet.numpy.io.CoreasShower` class has been introduced, which can take in an HDF5 file of any
   CoREAS shower. This class is a subclass of `smiet.numpy.io.Shower` as well as `smiet.numpy.io.CoreasHDF5`,
   and thus provides an interface that is similar to the `smiet.numpy.io.SlicedShower` class. This makes it
   easier to compare to/work with non-sliced CoREAS showers in analyses. The `smiet.numpy.io.SlicedShower`
   and `smiet.numpy.io.SlicedShowerCherenkov` classes are now subclasses of this new class.
2. To accommodate the new `CoreasShower` class, the `smiet.numpy.io` module has been completely refactored.
   All classes have been moved to submodules and are exposed through the `smiet.numpy.io` namespace.
3. The sorting of the antenna array of the `smiet.numpy.io.CoreasShower` class and subclasses has been changed
   to use the `np.lexsort()` function, which should be faster and more robust than the previous implementation.
4. The `smiet.numpy.geo_ce_to_e()` function has been fixed, since it had a wrong transformation from the
   GEO/CE traces to the vxvxB trace. Its signature has also been changed to accept the GEO and CE traces
   as two separate arguments, instead of a stacked array. This makes it easier to use in practice, since
   the `TemplateSynthesis` class returns them as two separate arrays.


JAX module
~~~~~~~~~~

1. To be consistent with the Numpy version, the `smiet.jax.io.CoreasHDF5` class has been renamed to
   `smiet.jax.io.CoreasShower`, since it provided similar functionality to the Numpy version's `CoreasShower`.
2. A method has been added to the `smiet.jax.synthesis.TemplateSynthesis` class to truncate the
   atmospheric grid, which essentially removes certain slices from the synthesis process.
3. The antenna sorting in the shower classes has been improved.

CORSIKA module
~~~~~~~~~~~~~~

1. When using the `smiet.corsika.write_simulation_to_file()`, the contents are again written to a
   subdirectory instead of the current working directory.
2. The `smiet.corsika.generate_simulation()` now accepts a `number_of_arms` argument, which allows
   to choose the number of arms for the star shape antenna array.
3. The `smiet.corsika.generate_simulation_from_hdf5()` has been made more flexible, allowing users
   to choose a different energy and primary particle type than the ones in the HDF5 file.