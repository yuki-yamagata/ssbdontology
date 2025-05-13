# SSBD Ontology (v2025‑05)

SSBD Ontology: A Two Tier Approach for Interoperable Bioimaging Metadata  
**Repository Layer** (rapid deposition) & **Added‑Value Layer** (curated rich annotations)

　Bioimaging data produced by modern microscopy are expanding at an unprecedented pace, yet their scientific value is often constrained by fragmented metadata. We present the SSBD Ontology, a two-layer semantic model that reconciles rapid publication with ontology-aligned curation. A lightweight repository layer captures only the indispensable descriptors required for DOI assignment, while an added-value layer enriches the same instances with biological context—organism, strain, cell type, anatomy and GO terms—as well as imaging-method and instrument semantics.

The ontology bridges the instance-centric nature of imaging repositories and the class-centric structure of existing OBO ontologies, enabling seamless integration into knowledge graphs. A single SPARQL query retrieves, across modalities, all datasets of C57BL/6J mouse brain recorded by AMATERAS light microscopy and FIB-SEM electron microscopy, returning direct OME-Zarr and Vizarr links in sub-second time.

SSBD Ontology (OWL DL), exemplar instances and conversion scripts are released under CC-BY 4.0 on GitHub and will be mirrored on BioPortal, strengthening the FAIR data ecosystem for bioimaging research.

---

## 1. Quick Overview (EN)

| Item | URL / File |
|------|------------|
| Core ontology (OWL/RDF‑XML) | [`ontology/ssbd_core.owl`](ontology/ssbd_core.owl) |
| All individuals (TTL) | [`data/ssbd_instances.ttl`](data/ssbd_instances.ttl) |
| Lineage diagram (Project 199) | ![](img/P199_ObjectGraph.svg) |
| Class hierarchy (Cell type, Anatomy, GO, etc.) | `img/` 配下 PNG/SVG |

### 1.1 Seven key entity types

| Layer | Entity class | Typical properties | Linked external vocab |
|-------|--------------|--------------------|-----------------------|
| **Repo** | `SSBD_Project` | `has_project_name`, `RO:0002234` (→ Dataset) | — |
| **Repo** | `SSBD_dataset` | `has_biosample_information`, `RO:0002180` (→ NGFF) | — |
| **Repo** | `SSBD_OME_NGFF_info` | `has_s3_endpoint`, `has_vizarr_url`, sizes | — |
| **Added** | `SSBD_biosample_information` | `is_about_organism/strain/cell/anatomy/GO*` | NCBITaxon, CL, UBERON, GO |
| **Added** | `SSBD_imaging_method_information` | `is_about_imaging_method` | FBbi |
| **Added** | `SSBD_imaging_instruments` | `has_component` (objective, detector …) | OBI / FBbi |
| **Added** | `SSBD_dimension_data` | x/y/z/t scale + unit | IAO / UO |

### 1.2 Example lineage (Project 199 – AMATERAS brain‑slice)

![](img/P199_ObjectGraph.svg)

*Project → Dataset → Biosample → OME‑Zarr* relations are encoded by  
`RO:0002234`, `has_biosample_information`, and `RO:0002180`.

### 1.3 Sample SPARQL query  
*(C57BL/6J strain & FIB‑SEM / AMATERAS datasets)*  
```sparql
PREFIX ssbdont: <http://ssbd.riken.jp/ontology/>
PREFIX obo:     <http://purl.obolibrary.org/obo/>

SELECT ?dataset ?title ?zarr ?vizarr
WHERE {
  ?bs ssbdont:is_about_strain <https://www.jax.org/strain/000664> .
  ?dataset ssbdont:has_biosample_information ?bs ;
           ssbdont:has_dataset_title ?title ;
           obo:RO_0002180 ?ngff ;
           ssbdont:has_imaging_method ?mNode .
  ?mNode ssbdont:is_about_imaging_method ?fbbi .
  FILTER(?fbbi IN (obo:FBbi_00000327, obo:FBbi_00000246))   # FIB‑SEM / AMATERAS
  ?ngff ssbdont:has_s3_endpoint ?zarr ;
        ssbdont:has_vizarr_url  ?vizarr .
}
