# CDIF Data Description Profile

Repository for the CDIF Data Description profile — extends the Discovery profile with detailed variable documentation and data structure description using DDI-CDI concepts.

## Specification

- **[CDIFDataDescriptionClasses.docx](CDIFDataDescriptionClasses.docx)** — Classes and properties documentation
- **[CDIFDataDescriptionProfileStructuredSchema.json](CDIFDataDescriptionProfileStructuredSchema.json)** — JSON Schema for validation (generated from metadataBuildingBlocks)
- **[rules.shacl](rules.shacl)** — SHACL validation shapes (synced from metadataBuildingBlocks)

## What Data Description adds to Discovery

The Data Description profile composes cdifCore + CDIFDiscoveryProfile + additional constraints:

- **`schema:variableMeasured`** items must have `@id` and are extended with DDI-CDI `cdi:InstanceVariable` properties (`cdi:physicalDataType`, `cdi:intendedDataType`, `cdi:role`, `cdi:uses`, `cdi:qualifies`)
- **`schema:distribution`** items gain `cdi:characterSet`, `cdi:fileSize`, `cdi:fileSizeUofM` for file characterization
- **`dcterms:conformsTo`** must include `https://w3id.org/cdif/data_description/1.0` in addition to core and discovery

## Examples

The `examples/` directory contains 6 validated JSON-LD examples at the DataDescription level (no provenance extensions):

| File | Domain | Variables | Key CDI content |
|------|--------|-----------|-----------------|
| `xas-na2so4-datadesc.json` | XAS spectroscopy | 3 | cdi:InstanceVariable, physicalDataType, uses, role |
| `xrd-datadesc.json` | X-ray diffraction | 2 | cdi:InstanceVariable, physicalDataType, uses |
| `general-datadesc.json` | GC-MS chemistry | 3 | cdi:InstanceVariable, physicalDataType |
| `nwis-longdata-datadesc.json` | Water quality | CDI in dist | Long format with cdi:LongStructureDataSet, CSVW |
| `cdif-datadesc-example.json` | General | 2 | cdi:InstanceVariable, measurement technique |
| `exampleCDIFDataDescription.json` | General | 2 | Building block reference example |

All validated against CDIFDataDescriptionProfile structured schema.

## Sources of variable-level metadata for examples

An investigation of repositories that could provide variable-level metadata for DataDescription examples (April 2026):

| Source | Variable-level DDI? | Notes |
|--------|---------------------|-------|
| **Harvard Dataverse** | **Yes** | DDI export (`/api/datasets/export?exporter=ddi`) includes full `<dataDscr>` with variable names, labels, formats, categories for ingested .tab files |
| **Borealis Dataverse** | No | DDI export available but no variable descriptions (shapefiles, not ingested tabular) |
| **SND / researchdata.se** | No | Study-level only in all formats: DDI Codebook 2.5, DDI Lifecycle 3.3, JSON-LD, REST API. Reports `variableQuantity` but doesn't expose variable list |
| **CESSDA Data Catalogue** | No | OAI-PMH DDI 2.5 is study-level only — no `<dataDscr>` section |
| **UK Data Service** | No | DDI via OAI-PMH has no `<dataDscr>` section |
| **GESIS** | Inaccessible | Has JSON-LD on landing pages but API blocked by SPA; direct DDI download returns empty |
| **ICPSR** | Inaccessible | Cloudflare bot protection blocks programmatic access |
| **PANGAEA** | Partial | schema.org JSON-LD on landing pages includes `variableMeasured` (names, units) but no DDI-CDI structure |
| **NCEI NOAA** | No | schema.org JSON-LD has spatialCoverage/temporalCoverage but no variable descriptions |
| **Kaggle** | No | Core-level schema.org only |

### Harvard Dataverse DDI export

The most promising source for variable-level DDI Codebook with `<dataDscr>`:

```
GET https://dataverse.harvard.edu/api/datasets/export?exporter=ddi&persistentId={doi}
```

Returns full DDI Codebook 2.5 XML including:
- `<var>` elements with `name`, `@intrvl` (discrete/contin)
- `<labl>` — variable labels
- `<varFormat>` — data type (numeric, character)
- `<catgry>` — category values for coded variables
- `<notes>` — UNF (Universal Numeric Fingerprint)
- `<location>` — file reference

Only available for datasets with ingested tabular files (`.tab` format in Dataverse). Datasets with non-tabular files (shapefiles, PDFs, etc.) return DDI without variable descriptions.

### SND / researchdata.se

Rich OAI-PMH endpoint with multiple formats but study-level only:

- **Endpoint**: `https://api.researchdata.se/oai-pmh`
- **Formats**: `oai_dc`, `ddi25`, `ddi33`, `json_ld`, `oai_datacite`, `cmdi`, `csl_json`
- **JSON-LD**: schema.org Dataset with spatialCoverage, temporalCoverage, structured DefinedTerm keywords from CESSDA vocabularies, ROR-identified affiliations
- **REST API**: `https://api.researchdata.se/dataset/{id}` returns dataset metadata + embedded JSON-LD
- **Limitation**: `variableQuantity: N` reported but actual variable list not exposed in any format

## Related repositories

- **[cdif-core](https://github.com/Cross-Domain-Interoperability-Framework/core)** — CDIF Core profile (base properties)
- **[Discovery](https://github.com/Cross-Domain-Interoperability-Framework/Discovery)** — CDIF Discovery profile (spatial, temporal, variables)
- **[data-structure-description](https://github.com/Cross-Domain-Interoperability-Framework/data-structure-description)** — DDI-CDI mapping development, example data files, and test metadata
- **[metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks)** — Building block schemas including cdifDataDescription, cdifVariableMeasured, cdifTabularData, cdifLongData, cdifDataCube
- **[validation](https://github.com/Cross-Domain-Interoperability-Framework/validation)** — Validation tools, DCAT converter, harvester

## License

See [LICENSE](LICENSE).
