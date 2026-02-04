# CF Conventions for Zarr

- **UUID**: 77c308c7-4db2-4774-8b2d-aa37e9997db6
- **Name**: CF
- **Schema URL**: https://raw.githubusercontent.com/zarr-conventions/CF/refs/tags/v1/schema.json
- **Spec URL**: https://cfconventions.org/Data/cf-conventions/cf-conventions-1.11/cf-conventions.html
- **Scope**: Array, Group
- **Extension Maturity Classification**: Proposal

## Description

This convention registers the use of [CF (Climate and Forecast) conventions](https://cfconventions.org/) in Zarr datasets. CF conventions are the de facto standard for encoding climate and forecast model data, originally developed for NetCDF but now widely used with Zarr through libraries like xarray.

**Key Design Decision**: Unlike other GeoZarr conventions that use namespace prefixes (e.g., `proj:`, `spatial:`), CF attributes use their **standard unprefixed names** for backwards compatibility with:

- Existing CF-compliant Zarr datasets
- Libraries like cf-python, xarray, and cfgrib
- NetCDF-to-Zarr converted datasets

The convention registration in `zarr_conventions` declares intent to use CF conventions, while actual CF attributes remain unprefixed.

### Examples

- [Minimal registration](examples/minimal.json) - Group with CF convention registration
- [Temperature variable](examples/temperature.json) - Data variable with standard_name, units
- [Coordinate variable](examples/coordinate_variable.json) - Latitude coordinate with axis, bounds
- [Time coordinate](examples/time_coordinate.json) - Time coordinate with calendar
- [Grid mapping](examples/grid_mapping.json) - CRS definition via grid_mapping attributes
- [Flag variable](examples/flag_variable.json) - Categorical data with flag_values/flag_meanings
- [Composed with proj:](examples/composed_with_proj.json) - CF + geo-proj convention
- [Composed with spatial:](examples/composed_with_spatial.json) - CF + spatial convention

## Motivation

- **Backwards compatibility**: Millions of CF-compliant datasets exist; this convention allows registering CF usage without modifying existing attributes
- **Library compatibility**: cf-python, xarray, and other CF-aware libraries can process data without modification
- **Version tracking**: The `version` field in the convention metadata allows tracking which CF version the dataset conforms to
- **Composability**: CF convention can be combined with geo-proj, spatial, and multiscales conventions

## Convention Registration

The convention must be registered in `zarr_conventions`:

```json
{
  "zarr_conventions": [
    {
      "schema_url": "https://raw.githubusercontent.com/zarr-conventions/CF/refs/tags/v1/schema.json",
      "spec_url": "https://cfconventions.org/Data/cf-conventions/cf-conventions-1.11/cf-conventions.html",
      "uuid": "77c308c7-4db2-4774-8b2d-aa37e9997db6",
      "name": "CF",
      "version": "1.11",
      "description": "Climate and Forecast conventions for self-describing geospatial data"
    }
  ]
}
```

The `version` field tracks the CF conventions version (e.g., "1.11", "1.10", "1.8").

## Applicable To

This convention can be used with these parts of the Zarr hierarchy:

- [x] Group (global attributes like `Conventions`, `title`, `institution`)
- [x] Array (variable attributes like `standard_name`, `units`, `coordinates`)

## Core CF Attributes

CF attributes are placed directly in `attributes` without any prefix. The full list of CF attributes is documented in the [CF Conventions specification](https://cfconventions.org/Data/cf-conventions/cf-conventions-1.11/cf-conventions.html).

### Global Attributes (Group Level)

| Field Name      | Type   | Description                                                  |
| --------------- | ------ | ------------------------------------------------------------ |
| Conventions     | string | CF conventions version string (e.g., "CF-1.11")              |
| title           | string | A succinct description of what is in the dataset             |
| institution     | string | Where the original data was produced                         |
| source          | string | The method of production of the original data                |
| history         | string | Audit trail for modifications to the original data           |
| references      | string | Published or web-based references                            |
| comment         | string | Miscellaneous information about the data                     |

### Variable Attributes (Array Level)

| Field Name          | Type            | Description                                              |
| ------------------- | --------------- | -------------------------------------------------------- |
| standard_name       | string          | Name from the CF Standard Name Table                     |
| long_name           | string          | Descriptive name for the variable                        |
| units               | string          | Units recognized by UDUNITS                              |
| axis                | enum            | Axis type: "X", "Y", "Z", or "T"                         |
| positive            | enum            | Vertical direction: "up" or "down"                       |
| calendar            | enum            | Calendar for time coordinates                            |
| coordinates         | string          | Space-separated list of coordinate variables             |
| bounds              | string          | Name of boundary variable                                |
| cell_methods        | string          | Statistical operations on cells                          |
| cell_measures       | string          | Variables containing cell areas/volumes                  |
| grid_mapping        | string          | Name of grid mapping variable                            |
| ancillary_variables | string          | Space-separated list of ancillary variables              |
| \_FillValue         | number/string   | Value indicating missing data                            |
| valid_min           | number          | Minimum valid value                                      |
| valid_max           | number          | Maximum valid value                                      |
| valid_range         | [number,number] | Two-element array [min, max]                             |
| scale_factor        | number          | Scale for packed data                                    |
| add_offset          | number          | Offset for packed data                                   |
| flag_values         | [integer]       | Array of flag values for categorical data                |
| flag_meanings       | string          | Space-separated flag meanings                            |
| flag_masks          | [integer]       | Array of flag masks for bit-field data                   |

### Grid Mapping Attributes

| Field Name                        | Type          | Description                         |
| --------------------------------- | ------------- | ----------------------------------- |
| grid_mapping_name                 | string        | Name of projection method           |
| crs_wkt                           | string        | WKT2 representation of CRS          |
| semi_major_axis                   | number        | Ellipsoid semi-major axis           |
| semi_minor_axis                   | number        | Ellipsoid semi-minor axis           |
| inverse_flattening                | number        | Ellipsoid inverse flattening        |
| earth_radius                      | number        | Spherical Earth radius              |
| false_easting                     | number        | False easting                       |
| false_northing                    | number        | False northing                      |
| longitude_of_central_meridian     | number        | Central meridian longitude          |
| latitude_of_projection_origin     | number        | Projection origin latitude          |
| longitude_of_projection_origin    | number        | Projection origin longitude         |
| scale_factor_at_central_meridian  | number        | Scale factor at central meridian    |
| scale_factor_at_projection_origin | number        | Scale factor at projection origin   |
| standard_parallel                 | number/array  | Standard parallel(s)                |

## Composition with Other Conventions

### CF + geo-proj

CF's `grid_mapping` attributes and geo-proj's `proj:code`/`proj:wkt2`/`proj:projjson` both encode CRS information. They can coexist and should be kept consistent:

```json
{
  "zarr_conventions": [
    { "name": "CF", "uuid": "77c308c7-4db2-4774-8b2d-aa37e9997db6", "version": "1.11" },
    { "name": "proj:", "uuid": "f17cb550-5864-4468-aeb7-f3180cfb622f" }
  ],
  "standard_name": "air_temperature",
  "units": "K",
  "grid_mapping": "crs",
  "proj:code": "EPSG:4326"
}
```

### CF + spatial

CF uses coordinate variables for spatial reference; the spatial convention uses explicit transforms. They complement each other:

```json
{
  "zarr_conventions": [
    { "name": "CF", "uuid": "77c308c7-4db2-4774-8b2d-aa37e9997db6", "version": "1.11" },
    { "name": "spatial:", "uuid": "689b58e2-cf7b-45e0-9fff-9cfc0883d6b4" }
  ],
  "standard_name": "sea_surface_temperature",
  "units": "K",
  "coordinates": "lat lon",
  "spatial:dimensions": ["lat", "lon"],
  "spatial:transform": [0.25, 0.0, -180.0, 0.0, -0.25, 90.0]
}
```

### CF + multiscales

CF doesn't define pyramid structures. The multiscales convention extends CF datasets:

```json
{
  "zarr_conventions": [
    { "name": "CF", "uuid": "77c308c7-4db2-4774-8b2d-aa37e9997db6", "version": "1.11" },
    { "name": "multiscales", "uuid": "d35379db-88df-4056-af3a-620245f8e347" }
  ],
  "Conventions": "CF-1.11",
  "multiscales": {
    "layout": [
      { "asset": "0", "transform": { "scale": [1, 1] } },
      { "asset": "1", "derived_from": "0", "transform": { "scale": [2, 2] } }
    ]
  }
}
```

## Migration Guide

### Adding CF Registration to Existing Datasets

Existing CF-compliant Zarr datasets can be registered without modifying CF attributes:

**Before:**

```json
{
  "Conventions": "CF-1.8",
  "standard_name": "air_temperature",
  "units": "K"
}
```

**After:**

```json
{
  "zarr_conventions": [
    { "name": "CF", "uuid": "77c308c7-4db2-4774-8b2d-aa37e9997db6", "version": "1.8" }
  ],
  "Conventions": "CF-1.8",
  "standard_name": "air_temperature",
  "units": "K"
}
```

## Known Implementations

### Libraries and Tools

- **[cf-python](https://ncas-cms.github.io/cf-python/)** - CF-compliant Python library
  - Language: Python
  - Status: Production
  - Since: 2016

- **[xarray](https://docs.xarray.dev/)** - N-D labeled arrays with CF support via cf-xarray
  - Language: Python
  - Status: Production

- **[cfgrib](https://github.com/ecmwf/cfgrib)** - CF-compliant GRIB to Zarr/NetCDF
  - Language: Python
  - Status: Production

## References

- [CF Conventions 1.11](https://cfconventions.org/Data/cf-conventions/cf-conventions-1.11/cf-conventions.html)
- [CF Standard Name Table](https://cfconventions.org/Data/cf-standard-names/current/build/cf-standard-name-table.html)
- [cf-python documentation](https://ncas-cms.github.io/cf-python/)
- [GeoZarr Conventions Specification](https://github.com/zarr-conventions/zarr-conventions-spec)

## Acknowledgements

This convention builds upon work from multiple communities and projects:

### Specification Sources

- **[CF Conventions](https://cfconventions.org/)** - The Climate and Forecast conventions maintained by the CF Governance Panel. CF conventions define the metadata attributes used in this specification.
- **[STAC Extensions Template](https://github.com/stac-extensions/template)** - The convention documentation structure is based on the STAC extensions template.
- **[geo-proj Convention](https://github.com/zarr-experimental/geo-proj)** and **[spatial Convention](https://github.com/zarr-conventions/spatial)** - Sister conventions that this CF convention is designed to compose with.

### External Standards

- **[UDUNITS](https://www.unidata.ucar.edu/software/udunits/)** - Unit handling library referenced by CF for units validation.
- **[OGC/ISO Standards](https://www.ogc.org/)** - WKT2 (ISO 19162:2019) for coordinate reference system representation in `crs_wkt`.

### Software Libraries

- **[cf-python](https://ncas-cms.github.io/cf-python/)** - CF-compliant Python library that informed attribute validation patterns.
- **[xarray](https://xarray.pydata.org/)** and **[cf-xarray](https://cf-xarray.readthedocs.io/)** - N-dimensional labeled arrays with CF support.
- **[netCDF4-python](https://unidata.github.io/netcdf4-python/)** - NetCDF library that established CF-on-NetCDF patterns now adapted for Zarr.

### Contributing Organizations

- **[Unidata](https://www.unidata.ucar.edu/)** - For maintaining CF conventions and related infrastructure.
- **[UK Met Office](https://www.metoffice.gov.uk/)** - For cf-python and CF conventions governance contributions.
