# CDIF Data Description Profile

Repository for the CDIF Data Description profile — extends the Discovery profile with detailed variable documentation and data structure description using DDI-CDI concepts.

## Specification

- **[CDIFDataDescriptionClasses.docx](CDIFDataDescriptionClasses.docx)** — Classes and properties documentation
- **[CDIFDataDescriptionProfileStructuredSchema.json](CDIFDataDescriptionProfileStructuredSchema.json)** — JSON Schema for validation (generated from metadataBuildingBlocks)
- **[rules.shacl](rules.shacl)** — SHACL validation shapes (synced from metadataBuildingBlocks)

## What Data Description adds to Discovery

The Data Description profile composes cdifCore + CDIFDiscoveryProfile + additional constraints:

- **`schema:variableMeasured`** items must have `@id` and `@type` including `cdi:InstanceVariable`, extended with DDI-CDI properties (`cdi:physicalDataType`, `cdi:intendedDataType`, `cdi:role`, `cdi:uses`, `cdi:qualifies`)
- **`schema:distribution`** items can be typed as `cdi:TabularTextDataSet`, `cdi:LongStructureDataSet`, or `cdi:StructuredDataSet` with CSVW properties (`csvw:delimiter`, `csvw:header`, `csvw:headerRowCount`) and dimension counts (`cdifq:nRows`, `cdifq:nColumns`)
- **`cdi:hasPhysicalMapping`** on distributions links physical columns to `variableMeasured` items via `cdi:formats_InstanceVariable` references with column indices and physical data types
- **File characterization**: `cdi:characterSet`, `cdi:fileSize`, `cdi:fileSizeUofM`, `schema:contentSize`, `spdx:checksum`
- **`dcterms:conformsTo`** must include `https://w3id.org/cdif/data_description/1.0` in addition to core and discovery

## Examples

The `examples/` directory contains 7 validated JSON-LD examples at the DataDescription level. All use DataDescription-specific properties (not just Discovery-level metadata). Records that only had `cdi:InstanceVariable` typing without physical data types or mappings have been moved to Discovery or packaging repositories.

### DDI Codebook conversions (Harvard Dataverse)

Converted from DDI Codebook 2.5 XML via `validation/DDI/ddi_to_cdif.py`. Tab file headers fetched from Dataverse to build physical mappings. File sizes and MD5 checksums from Dataverse file API.

| File | Domain | Vars | Files | Rows | DD properties |
|------|--------|------|-------|------|---------------|
| `ddi-cost-of-cash-datadesc.json` | Economics | 75 | 3 | 1,256 | intendedDataType, role, hasPhysicalMapping, TabularTextDataSet, csvw, nRows/nColumns, checksum |
| `ddi-expert-party-space-datadesc.json` | Political science | 74 | 2 | 401 | intendedDataType, role, hasPhysicalMapping, TabularTextDataSet, csvw, nRows/nColumns, checksum |
| `ddi-zoonotic-disease-datadesc.json` | Climate & disease | 7 | 1 | 315 | intendedDataType, role, hasPhysicalMapping, TabularTextDataSet, csvw, nRows/nColumns, checksum |
| `ddi-bird-monitoring-datadesc.json` | Biodiversity | 82 | 5 | 2,429 | intendedDataType, role, hasPhysicalMapping, TabularTextDataSet, csvw, nRows/nColumns, checksum |

### Simplified from validation MetadataExamples

Provenance extensions stripped; retain only DataDescription-specific properties.

| File | Domain | DD properties |
|------|--------|---------------|
| `cdif-datadesc-example.json` | General | physicalDataType, hasPhysicalMapping, TabularTextDataSet, StructuredDataSet, csvw |
| `nwis-longdata-datadesc.json` | Water quality | physicalDataType, role, hasPhysicalMapping, LongStructureDataSet, csvw |
| `xas-na2so4-datadesc.json` | XAS spectroscopy | physicalDataType, uses |

All validated against CDIFDataDescriptionProfile structured schema.

## Sources of variable-level metadata

Investigation of repositories for variable-level metadata (April 2026):

| Source | Variable-level DDI? | Notes |
|--------|---------------------|-------|
| **Harvard Dataverse** | **Yes** | DDI export includes full `<dataDscr>` with variable names, labels, formats, categories for ingested .tab files |
| **Borealis Dataverse** | No | DDI export available but no variable descriptions |
| **SND / researchdata.se** | No | Study-level only in all formats (DDI 2.5, DDI 3.3, JSON-LD). Reports `variableQuantity` but doesn't expose variable list |
| **CESSDA Data Catalogue** | No | OAI-PMH DDI 2.5 is study-level only |
| **UK Data Service** | No | DDI via OAI-PMH has no `<dataDscr>` section |
| **GESIS** | Inaccessible | API blocked by SPA |
| **ICPSR** | Inaccessible | Cloudflare bot protection |
| **PANGAEA** | Partial | `variableMeasured` (names, units) but no DDI-CDI structure |

### Harvard Dataverse DDI export

```
GET https://dataverse.harvard.edu/api/datasets/export?exporter=ddi&persistentId={doi}
```

Returns DDI Codebook 2.5 with `<dataDscr>` including `<var>` elements (name, label, format, interval, summary statistics, file reference) and `<fileDscr>` with dimensions (`caseQnty`, `varQnty`). Only for datasets with ingested .tab files.

### Conversion tool

`validation/DDI/ddi_to_cdif.py` converts DDI Codebook to CDIF DataDescription JSON-LD:

```bash
python DDI/ddi_to_cdif.py input.xml --doi https://doi.org/10.7910/DVN/XXX \
  --fetch-headers --fetch-file-meta -o output.json
```

## JSON-LD Framing and Validation

**`FrameAndValidate.py`** frames a JSON-LD document against the Data Description profile schema and optionally validates it:

```bash
# Frame and validate
python FrameAndValidate.py examples/cdif-datadesc-example.json --validate

# Frame and save output
python FrameAndValidate.py examples/cdif-datadesc-example.json -o framed.json
```

The script uses **`CDIFDataDescription-frame.jsonld`** to frame JSON-LD documents into the expected property structure. Context prefixes from the input document are automatically merged into the frame.

**Requirements:** `pyld`, `jsonschema` (`pip install pyld jsonschema`)

## SHACL Validation

**`rules.shacl`** contains self-contained SHACL shapes for validating CDIF Data Description profile instances. This file merges shapes from all composing building blocks (cdifCore, cdifCatalogRecord, cdifDataDescription, definedTerm, variableMeasured, spatialExtent, temporalExtent, qualityMeasure) with the profile-level shapes, so it can be used standalone. Source shapes come from [`metadataBuildingBlocks/_sources/`](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks/tree/main/_sources) and should be regenerated when source shapes change.

## Related repositories

- **[cdif-core](https://github.com/Cross-Domain-Interoperability-Framework/core)** — CDIF Core profile
- **[Discovery](https://github.com/Cross-Domain-Interoperability-Framework/Discovery)** — CDIF Discovery profile
- **[packaging](https://github.com/Cross-Domain-Interoperability-Framework/packaging)** — Archive bundles and RO-Crate (records moved here that are bundles without DD properties)
- **[data-structure-description](https://github.com/Cross-Domain-Interoperability-Framework/data-structure-description)** — DDI-CDI mapping development and test data
- **[metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks)** — Building block schemas
- **[validation](https://github.com/Cross-Domain-Interoperability-Framework/validation)** — Validation tools, DDI converter, DCAT converter, harvester

## License

See [LICENSE](LICENSE).
