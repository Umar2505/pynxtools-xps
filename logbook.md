### Entry 001
- **Date:** 2026-06-22
- **Change / Issue Type:** completion of metadata parser for phi vendor .spe file
- **Title:** Adapting metadata parser for the specific unit kV
- **Error message:**
```bash
pint.errors.UndefinedUnitError: 'Invalid unit 'KV_Cr' at path: /ENTRY[C1s]/INSTRUMENT[instrument]/source_probe/voltage/@units' is not defined in the unit registry
```
- **Reason for the error:**

The function inside parsers/phi/metadata.py _convert_xray_source_settings() is not implemented to fully normalize units from vendor which causes inconsistencies with the _UNIT_MAP. Specifically, the code expects kV as a unit but recieves KV_Cr.

- **Affected file(s):** src/pynxtools_xps/parsers/phi/metadata.py
- **Function / method name:** created new func inside _convert_xray_source_settings() named _normalize_xray_unit()
- **Original function:**

```python
def _convert_xray_source_settings(value: str):
    """Map all items in xray_source_settings to a dictionary."""
    (xray_settings) = re.split(r"(\d+)", value)

    # TODO: This should be done through the _context
    # # for i, setting in enumerate(xray_settings):
    #     xray_settings[i] = convert_units(setting)

    return {
        "spot_size": float(xray_settings[1]),
        "spot_size_units": xray_settings[2],
        "power": float(xray_settings[3]),
        "power_units": xray_settings[4],
        "high_voltage": float(xray_settings[5]),
        "high_voltage_units": xray_settings[6],
    }
```

- **Solution implemented:**

Implemented simple normalization function that splits the unit string by the underscore and turns 'KV' into 'kV'.

- **Logic behind the solution:**

From the given error message I got an invalid unit that is being used and wrote code that would make it alligned with the unit map _UNIT_MAP

- **Updated function code:**

```python
def _convert_xray_source_settings(value: str):
    """Map all items in xray_source_settings to a dictionary."""

    def _normalize_xray_unit(unit: str) -> str:
        """Normalize vendor-specific units while preserving the physical unit."""
        base_unit = unit.split("_", 1)[0]
        if base_unit.upper() == "KV":
            return "kV"
        return unit

    xray_settings = re.split(r"(\d+)", value)

    return {
        "spot_size": float(xray_settings[1]),
        "spot_size_units": _normalize_xray_unit(xray_settings[2]),
        "power": float(xray_settings[3]),
        "power_units": _normalize_xray_unit(xray_settings[4]),
        "high_voltage": float(xray_settings[5]),
        "high_voltage_units": _normalize_xray_unit(xray_settings[6]),
    }
```

- **Verification / tests run:** I did not write any tests but now it successfully converts .spe files into valid .nxs file
- **Short summary:** The project now can normalize vendor specific units (like 'KV_Cr') into valid 'kV' unit