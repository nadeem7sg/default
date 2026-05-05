# Sherpa default-repo fixtures

This repository is the **central fixture set** for the Sherpa `default_repo`
DAG.  Each branch represents one specific scenario the pipeline should
detect.

| Branch | Scenario | Where it gates |
|---|---|---|
| `main` | happy path — clean Azure + onprem TDF, two well-formed DSDFs | `completed`, all promoted |
| `tdf-missing-version` | TDF with no `metadata.version` | `default_repo_tdf_schema_validator` (`required`) |
| `tdf-bad-apiversion` | TDF with malformed `apiVersion` | schema (`pattern`) |
| `tdf-wrong-kind` | `kind:` is not `TargetDefinition` | schema (`const`) |
| `tdf-bad-name-slug` | `metadata.name: Azure_East` | schema (`pattern`) |
| `tdf-empty-regions` | `target.regions: []` | schema (`minItems`) |
| `tdf-no-services` | `target.services: []` | schema (`minItems`) |
| `tdf-bad-service-type` | service type not in enum | schema (`enum`) |
| `tdf-malformed-yaml` | broken YAML | yaml-validator + parse-error |
| `tdf-empty-file` | 0-byte `.yaml` | parse-error |
| `tdf-bad-region` | `regions: ["us-east-99"]` | `default_repo_cloud_resource_checker` |
| `tdf-placeholder-name` | `metadata.name: TODO` | content checker |
| `tdf-placeholder-region` | `regions: ["<your-region>"]` | content checker |
| `dsdf-missing-tags-env` | DSDF without `metadata.tags.env` | schema (`required`) |
| `dsdf-bad-env` | `env: production` (not in enum) | schema (`enum`) |
| `dsdf-bad-connection-type` | `connection.type: blob-store` | schema (`enum`) |
| `dsdf-bad-secrets-provider` | `secrets.provider: bitwarden` | schema (`enum`) |
| `dsdf-vector-store-no-vectorindex` | vector-store without `vectorIndex` block | schema (conditional `required`) |
| `dsdf-vectorindex-bad-dims` | `vectorIndex.dims: 0` | schema (`minimum`) |
| `dsdf-vector-store-valid` | vector-store WITH valid `vectorIndex` (positive) | promotes |
| `dsdf-orphan-tdfref` | `home.tdfRef` points to non-existent TDF | cross-ref check |
| `partial-dsdf-only` | DSDF with no matching TDF in repo | cross-ref → all rejected |
| `mixed-pass-fail` | one valid TDF + one invalid TDF | `completed` with mixed counts |
| `name-collision` | two files with same `metadata.name` + version | writeback `duplicate_count` |
| `empty-repo` | only `README.md`, no YAML | intake → 0 discovered |
| `no-tdf-dsdf` | YAMLs with `kind: WorkloadDefinition` only | intake → 0 discovered |
| `wdf-only-ignored` | only WDF files | intake skips → 0 discovered |
| `tdf-additionalprops-typo` | typo'd top-level key (`targetz:`) — false positive | schema PASSES |
| `tdf-deprecated-apiversion` | `tdf.7sg.ai/v99` — false negative | schema PASSES |

## Conventions

- TDF files use `*.tdf.yaml`; DSDFs use `*.dsdf.yaml`.
- `metadata.name` slugs are `[a-z][a-z0-9-]{1,62}`.
- DSDF.`home.tdfRef` must match a sibling TDF's `metadata.name`.
- The pipeline only inspects YAML files; non-YAML and `kind: WorkloadDefinition`
  documents are silently skipped at intake.
