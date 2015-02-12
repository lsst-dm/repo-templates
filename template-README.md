LSST org repo template README
=============================

Overview
--------

The `pex_config` package provides for configurations for the LSST Data 
Management System.

Configurations are hierarchical trees of parameters used to control the
execution of code.  They should not be confused with input data.
`pex_config` configurations (`Config` objects) are validatable,
documentable, and recordable.  The configuration parameters are stored in
attributes (member variables) of type `Field` in the `Config` object. Their
values can be recorded for provenance purposes. The history of their changes
during program execution is also tracked, and can be recorded and examined for
provenance and debugging purposes.

`pex_config` is implemented in Python.  Python code is used both in defining
the available `Field`s and in setting or modifying their values.  This
provides great flexibility and introspection while still being approachable to
non-programmers who need to modify configuration settings.

### Example

Defining a configuration subclass
```
import lsst.pex.config as pexConfig

class IsrTaskConfig(pexConfig.config):
  doWrite = pexConfig.Field("Write output?", bool, True)
  fwhm = pexConfig.Field("FWHM of PSF (arcsec)", float, 1.0)
  saturatedMaskName = pexConfig.Field(
      "Name of mask plane to use in saturation detection", str, "SAT")
  flatScalingType = pexConfig.ChoiceField(
      "The method for scaling the flat on the fly.", str,
      default='USER',
      allowed={
          "USER": "User defined scaling",
          "MEAN": "Scale by the inverse of the mean",
          "MEDIAN": "Scale by the inverse of the median",
      })
  keysToRemoveFromAssembledCcd = pexConfig.ListField(
      "fields to remove from the metadata of the assembled ccd.",
      dtype=str, default=[])
```
Here is a sample configuration override file to be used with the partial
``IsrTaskConfig`` above.  Note that all field names are prefixed with "root"
and that overrides should never be used to set already-existing default
values
```
root.doWrite = False
root.fwhm = 0.8
root.saturatedMaskName = 'SATUR'
root.flatScalingType = 'MEAN'
root.keysToRemoveFromAssembledCcd = ['AMPNAME']
```

### Typical usage
```
 def doIsrTask(ccd, configOverrideFilename=None):
     config = IsrTaskConfig()
     if configOverrideFilename is not None:
         config.load(configOverrideFilename)
         # Note: config override files are Python code with .py extensions
         # by convention
     config.validate()
     config.freeze()

     detectSaturation(ccd, config.fwhm, config.saturatedMaskName)
     # Note: methods typically do not need the entire config; they should be
     # passed only relevant parameters
     for k in config.keysToRemoveFromAssembledCcd:
         ccd.metadata.remove(k)
     if config.doWrite:
         ccd.write()
```

Details
-------

`pex_config` arose from a desire to have a configuration object holding
key-value pairs that also allows for (arbitrarily simple or complex) validation
of configuration values.

To configure code using `pex_config`, a developer subclasses the `Config`
class. The subclass definition specifies the available `Field`\ s, their
default values (if any), and their validation, if necessary.

`Config`\ s are hierarchical (see `ConfigField`), so calling code can embed
the configuration definitions of called code.

Configurations are *not* input data.  They should not be used in place of
function or method arguments, nor are they intended to replace ordinary
dictionary data structures.  A good rule of thumb is that if a particular
parameter does not have a useful default, it is probably an input rather than a
configuration parameter.  Another rule of thumb is that configuration
parameters should generally not be set in algorithmic code, only in
initialization or user interface code.

A `Config` subclass is instantiated to create a configuration object.  If any
default `Field` values need to be overridden, they can be set by assignment
to the object's `Field` attributes (e.g. `config.param1 = 3.14`), often in
a parent `Config`'s `setDefaults()` method, or by loading from an external
file.  The code then uses the configuration values by accessing the object's
`Field` attributes (e.g., `x = config.param1`).

The `Config` object can also be frozen; attempting to change the field
values of a frozen object will raise an exception. This is useful to 
expose bugs that change configuration values after none should happen.

Finally, the contents of `Config` objects may easily be dumped, for
provenance or debugging purposes.

<!-- #live-link-doc -->

Tested platforms
-----------

<!-- #live-link-test -->


Support
-------

For help please email dm-users@lists.lsst.org


Contributing
------------

Pull requests welcome.

<!-- #live-link-contribute -->

See Also
--------


