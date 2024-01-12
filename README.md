# Simulation Model - Development Implementation

Reference implementation of CTA simulation model parameters consisting of:

- [schema](schema) descriptions of all simulation model parameters
- [verified_model](verified_model) parameters. Parameters are verified using above schema.
- [validated_model](validated_model) parameters. Parameters are used in [simtools](https://github.com/gammasim/simtools).

## Verified and Validated Model

The verified model consists of a set of json or yaml files verified to follow the corresponding schema file. The simtools application `simtools-validate-file-using-schema` is used for the verification.

Validated parameters are used successfully in [simtools](https://github.com/gammasim/simtools) (this needs to be documented).
Parameters which are validated can be submitted to the simulation model database.

### Implementation

A typical model files includes:

```yaml
value:
type:
units:
metadata: { }
```

The `value` field contains the actual value of the parameter or points towards a data file (e.g., an astropy table in ecsv format).
(The inclusion of the unit and type in the model file is still under discussion since it is technically part of the schema and does not have to be included here explicitly.)

### Subdivision of model files for telescope types and sites

Model parameter files are sorted according to the telescope type and site (e.g., MSTN includes all model parameters relevant for the MST on the Northern site). The [Default](validated_model/Default) directory contains all model parameters with their default values.

## Schema files

Schema files describe simulation model parameters including (among others fields) name, type, format, applicable telescopes, and parameter description.
They also include information about setting and validation activities, data sources, and simulation software.
The schema files and especially the model parameter descriptions are derived from (and planned to be synchronized with) the [sim_telarray manual](https://www.mpi-hd.mpg.de/hfm/~bernlohr/sim_telarray/).

The files are in human readable yaml format and follow a fixed [json-schema](https://json-schema.org/).
The full description of the schema is in [data.metaschema.yml](https://github.com/gammasim/simtools/blob/main/simtools/schemas/data.metaschema.yml) (found in simtools meta_schema directory).

### Typical Example

The following example is for a single parameter description.

```yaml
%YAML 1.2
---
title: Schema for num_gains model parameter
version: 0.1.0
base_schema: simpipe-schema
base_schema_url:https://raw.githubusercontent.com/gammasim/simtools/main/simtools/schemas/data.metaschema.yml
base_schema_version: 0.1.0
name: num_gains
description: Number of different gains the input signal gets digitized.
data:
  - type: int
    units: dimensionless
    default: 1
    allowed_range:
      min: 1
      max: 2
instrument:
  class: Camera
  type:
    - LSTCam
    - FlashCam
    - NectarCam
    - SSTCam
    - SCTCam
activity:
  setting:
    - SetParameterFromExternal
  validation:
    - ValidateParameterByExpert
source:
  - Initial instrument setup
simulation_software:
  - name: sim_telarray
```

### Valid Keys

Valid keys are described in detail in [jsonschema.yml](./jsonschema.yml). The following list gives a short overview.

#### Header section

- `title`: Title of the schema file.
- `version`: Version of this schema file.
- `base_schema`: Name of the base schema.
- `base_schema_url`: URL of the base schema.
- `base_schema_version`: Version of the base schema.

#### Parameter description

- `name`: Name of the parameter.
- `description`: Description of the parameter.
- `short_description`: Short description of the parameter (optional).

#### Parameter data

The `data` field is used to describe the actual type, format, and allowed values of the parameter.

- `type`: Data type of the parameter.
- `units`: Units of the parameter (compatible with astropy units). Note that units are explicitly listed in the jsonschema.yml file.
- `default`: Default value of the parameter.
- `allowed_range`: Allowed range of the parameter.
- `allowed_values`: Allowed values of the parameter. Use if values are not continuous and restricted to a small list of allowed values.

Tables of any dimensions can be described using the `datatable` field. Example:

```yaml
data:
  - type: datatable
    table_columns:
      - name: wavelength
        description: Wavelength.
        required: true
        units: nm
        type: double
        required_range:
          min: 300.0
          max: 700.0
        input_processing:
          - remove_duplicates
          - sort
      - name: photo_detection_efficiency
        description: Average quantum or photon detection efficiency.
        required: true
        units: dimensionless
        type: double
        allowed_range:
          min: 0.0
          max: 1.0
```

Input processing in form of sorting, removing of duplicates, etc. can be specified using the `input_processing` field.

#### Instrument description

- `class`: Instrument class. Allow values are *Camera*, *Site*, *Structure*, *Telescope*
- `type`: Instrument type (e.g., *LSTCam*, *MSTStructure*); see [jsonschema.yml](./jsonschema.yml) for a complete list of allowed values.
- `site`: CTAO site. Allowed values are *North* and *South*.

### Activity description

Describes setting and validation activities. Each activity corresponds to a workflow as described in ../[workflows](../workflows).

### Data source description

Describes the source of the data or parameter (e.g., *Calibration*)

### Simulation software description

Describes the simulation software (e.g., *sim_telarray* or *corsika*) the parameter is used for.
For some parameters, different names are used in the different software packages (e.g. `corsika_observation_level` is `OBSLEVEL` in CORSIKA).
In this case, the 'parameter_name' field documents this mapping:

```yaml
simulation_software:
  - name: corsika
    parameter_name: OBSLEV
```
