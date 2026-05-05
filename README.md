# Sherpa `default_repo` fixture branches

This repository is the central fixture set for the Sherpa `default_repo` DAG.
Each branch is a deterministic test input that isolates **one specific
validation outcome** the pipeline should produce. **`main`** is the pristine
happy path; every other branch is a single, targeted mutation off of it.

- 1 `main` happy-path branch
- 31 schema-violation branches (one error per branch, distinct `rule`)
- 11 cloud / content / cross-ref branches (schema PASSES, downstream gate FAILS)
- 14 positive variants (multi-cloud, multi-connection, edge providers, etc.)
- 7 structural / intake-level branches (empty, only-WDF, multi-doc, etc.)
- 4 false-positive / false-negative branches (look broken but pass, or vice versa)

Total: **93 branches**, each verifiably hitting exactly one rule.

## Pipeline gate map

```
default_repo_preflight                   <- registration-level (branch / PAT)
└─ default_repo_intake                   <- file discovery (any *.yaml/*.yml)
   └─ lint_check + yaml_validation       <- parse + lint
      └─ default_repo_tdf_schema_validator   ┐
      └─ default_repo_dsdf_schema_validator  ├─ run per file
      └─ default_repo_tdf_content_checker    ┤
      └─ default_repo_dsdf_content_checker   ┤
      └─ default_repo_cloud_resource_checker ┘
         └─ default_repo_writeback_and_finalize  <- cross_ref + duplicates + status
```

## Happy path

| Branch | Files | Notes |
|---|---|---|
| `main` | `azure-east.tdf.yaml`, `local-dev.tdf.yaml`, `taxi-data.dsdf.yaml`, `llm-endpoint.dsdf.yaml` | clean Azure + onprem TDFs, two DSDFs with valid `home.tdfRef` |

## TDF schema — `required` (one missing field per branch)

| Branch | Missing field | Rule |
|---|---|---|
| `tdf-missing-apiversion` | `apiVersion` | `required` |
| `tdf-missing-kind` | `kind` | `required` |
| `tdf-missing-metadata` | entire `metadata` block | `required` |
| `tdf-missing-target` | entire `target` block | `required` |
| `tdf-missing-metadata-name` | `metadata.name` | `required` |
| `tdf-missing-version` | `metadata.version` | `required` |
| `tdf-missing-metadata-tags` | `metadata.tags` | `required` |
| `tdf-missing-target-provider` | `target.provider` | `required` |
| `tdf-missing-target-regions` | `target.regions` | `required` |
| `tdf-missing-target-services` | `target.services` | `required` |
| `tdf-service-missing-name` | `target.services[0].name` | `required` |
| `tdf-service-missing-type` | `target.services[0].type` | `required` |

## TDF schema — `pattern` / `const` / `enum` / `minItems` / `type`

| Branch | Mutation | Rule |
|---|---|---|
| `tdf-bad-apiversion` | `apiVersion: tdf.7sg.ai/vX` | `pattern` |
| `tdf-wrong-kind` | `kind: TargetDef` | `const` |
| `tdf-bad-name-slug` | `metadata.name: Azure_East` | `pattern` |
| `tdf-name-too-long` | 71-char name | `pattern` |
| `tdf-name-leading-digit` | `metadata.name: 1azure-east` | `pattern` |
| `tdf-name-leading-hyphen` | `metadata.name: -azure-east` | `pattern` |
| `tdf-bad-version-no-patch` | `metadata.version: "1.0"` | `pattern` |
| `tdf-bad-version-with-v` | `metadata.version: "v1.0.0"` | `pattern` |
| `tdf-bad-version-extra-segment` | `metadata.version: "1.0.0.0"` | `pattern` |
| `tdf-empty-regions` | `target.regions: []` | `minItems` |
| `tdf-no-services` | `target.services: []` | `minItems` |
| `tdf-bad-service-type` | service `type: k8s-cluster` | `enum` |
| `tdf-regions-not-array` | `target.regions: eastus` (string) | `type` |
| `tdf-services-not-array` | `target.services: kubernetes` | `type` |

## TDF parse / structural

| Branch | Mutation | Rule |
|---|---|---|
| `tdf-malformed-yaml` | indentation broken mid-doc | `parse` |
| `tdf-empty-file` | 0-byte `.yaml` | `parse` (empty) |
| `non-mapping-root` | YAML root is a list | `type` |
| `multi-doc-yaml` | two YAML docs separated by `---` | `parse` (extra doc) |

## TDF cloud resource check (schema PASSes)

| Branch | Mutation | Gate |
|---|---|---|
| `tdf-bad-region` | `regions: ["us-east-99"]` (azure) | `cloud_check` |
| `tdf-bad-region-aws` | provider `aws`, region `us-east-99` | `cloud_check` |
| `tdf-bad-region-azure` | region `eastus-fake` | `cloud_check` |
| `tdf-bad-region-gcp` | provider `gcp`, region `us-fake-1` | `cloud_check` |
| `tdf-unknown-provider` | provider `oracle-cloud` | `cloud_check` |

## TDF content placeholders (schema may PASS)

| Branch | Mutation | Gate |
|---|---|---|
| `tdf-placeholder-name` | `metadata.name: TODO` | schema `pattern` + content |
| `tdf-placeholder-region` | `regions: ["<your-region>"]` | content placeholder |
| `tdf-placeholder-fixme-version` | `metadata.version: "FIXME"` | schema `pattern` + content |
| `tdf-placeholder-changeme-provider` | `provider: CHANGEME` | content placeholder |
| `tdf-placeholder-tbd-region` | `regions: ["TBD"]` | content placeholder |
| `tdf-empty-string-provider` | `provider: ""` | content (empty string) |

## DSDF schema — `required` (root + nested)

| Branch | Missing field | Rule |
|---|---|---|
| `dsdf-missing-apiversion` | `apiVersion` | `required` |
| `dsdf-missing-kind` | `kind` | `required` |
| `dsdf-missing-metadata` | `metadata` | `required` |
| `dsdf-missing-connection` | `connection` | `required` |
| `dsdf-missing-secrets` | `secrets` | `required` |
| `dsdf-missing-name` | `metadata.name` | `required` |
| `dsdf-missing-version` | `metadata.version` | `required` |
| `dsdf-missing-tags` | `metadata.tags` | `required` |
| `dsdf-missing-tags-env` | `metadata.tags.env` | `required` |
| `dsdf-missing-connection-type` | `connection.type` | `required` |
| `dsdf-missing-connection-provider` | `connection.provider` | `required` |
| `dsdf-missing-secrets-provider` | `secrets.provider` | `required` |
| `dsdf-missing-secrets-secretrefs` | `secrets.secretRefs` | `required` |
| `dsdf-secretref-missing-name` | `secrets.secretRefs[0].name` | `required` |
| `dsdf-secretref-missing-injectas` | `secrets.secretRefs[0].injectAs` | `required` |
| `dsdf-vector-store-no-vectorindex` | `vectorIndex` (conditional) | `required` |

## DSDF schema — `pattern` / `const` / `enum` / `minimum`

| Branch | Mutation | Rule |
|---|---|---|
| `dsdf-bad-apiversion` | `dsdf.7sg.ai/vX` | `pattern` |
| `dsdf-bad-version-format` | `metadata.version: "1.0"` | `pattern` |
| `dsdf-wrong-kind` | `kind: DataSource` | `const` |
| `dsdf-bad-env` | `tags.env: production` | `enum` |
| `dsdf-bad-connection-type` | `connection.type: blob-store` | `enum` |
| `dsdf-bad-secrets-provider` | `secrets.provider: bitwarden` | `enum` |
| `dsdf-bad-auth-method` | `auth.method: ssh` | `enum` |
| `dsdf-bad-vectorindex-metric` | `vectorIndex.metric: euclidean` | `enum` |
| `dsdf-secretref-bad-injectas` | `secretRefs[0].injectAs: envvar` | `enum` |
| `dsdf-vectorindex-bad-dims` | `vectorIndex.dims: 0` | `minimum` |

## Cross-document / writeback

| Branch | Mutation | Gate |
|---|---|---|
| `dsdf-orphan-tdfref` | `home.tdfRef` points to non-existent TDF | writeback `cross_ref` |
| `partial-dsdf-only` | repo has DSDFs only, no matching TDF | `cross_ref` → all rejected |
| `mixed-pass-fail` | one valid TDF + one invalid TDF | run completes with mixed counts |
| `name-collision` | two TDF files with identical `metadata.name`+version | writeback `duplicate_count` |
| `tdf-and-dsdf-same-name` | TDF and DSDF share `metadata.name` (positive) | both promote |

## Intake-level (no schema validation possible)

| Branch | Repo content | Outcome |
|---|---|---|
| `empty-repo` | only `README.md` | `failure_step: intake`, 0 discovered |
| `no-tdf-dsdf` | only `kind: WorkloadDefinition` YAML | 0 TDF/DSDF discovered |
| `wdf-only-ignored` | only a WDF file | default-repo intake silently drops |

## Positive variants

| Branch | Scenario |
|---|---|
| `dsdf-vector-store-valid` | vector-store DSDF with valid `vectorIndex` |
| `tdf-aws-happy` | AWS provider, regions `us-east-1`/`us-east-2` |
| `tdf-gcp-happy` | GCP provider, regions `us-central1`/`us-east1` |
| `tdf-edge-skip` | provider `edge` — cloud_check skipped |
| `tdf-custom-skip` | provider `custom` — cloud_check skipped |
| `tdf-multiple-regions` | 5 valid Azure regions |
| `dsdf-database-happy` | `connection.type: database` |
| `dsdf-queue-happy` | `connection.type: queue` |
| `dsdf-filesystem-happy` | `connection.type: filesystem` |
| `dsdf-object-store-happy` | `connection.type: object-store` |
| `dsdf-vault-secrets-happy` | `secrets.provider: vault` with populated `secretRefs` |

## Structural

| Branch | What it tests |
|---|---|
| `nested-subdirectory` | TDF/DSDF under `tdfs/` and `dsdfs/` subfolders |
| `yml-extension` | files use `.yml` instead of `.yaml` |

## False positive / false negative

| Branch | Why interesting |
|---|---|
| `tdf-additionalprops-typo` | typo'd `targetz:` block — schema accepts it (FALSE POSITIVE) |
| `dsdf-additionalprops-typo` | typo'd `connectionz:` block — schema accepts it |
| `tdf-deprecated-apiversion` | `tdf.7sg.ai/v99` matches pattern (FALSE NEGATIVE) |
| `dsdf-deprecated-apiversion` | `dsdf.7sg.ai/v99` matches pattern |

## Schema reference

Each fixture is built against:

- **TDF**: `assets/wdf-tdf-dsdf-package/schema/tdf.schema.202602191817.json`
- **DSDF**: `assets/wdf-tdf-dsdf-package/schema/dsdf.schema.202602191817.json`

Required field summary:

```
TDF root        : apiVersion, kind, metadata, target
TDF metadata    : name, version, tags
TDF target      : provider, regions (>=1), services (>=1)
TDF service[i]  : name, type (enum: kubernetes|serverless|spark|ml-platform|
                  object-store|file-store|database|vector-store|queue|
                  observability|secrets|identity|network|custom|container-apps|
                  container-platform|container-registry)

DSDF root       : apiVersion, kind, metadata, connection, secrets
DSDF metadata   : name, version, tags (with `env` in {dev,test,staging,prod})
DSDF connection : type (enum: object-store|database|warehouse|vector-store|
                  queue|api|filesystem), provider
DSDF secrets    : provider (enum: inherit|vault|awsSecretsManager|
                  gcpSecretManager|azureKeyVault|kubernetes|container-apps|
                  container-platform|container-registry|external),
                  secretRefs (each with name + injectAs in {env,file})
DSDF auth.method: enum: oidc|aws-iam-role|gcp-workload-identity|
                  azure-managed-identity|apikey|password|oauth2-client|
                  mtls|none
DSDF vectorIndex: required if connection.type == vector-store
                  dims >= 1, metric enum: cosine|dot|l2
```

## Slug pattern

`metadata.name` must match `^[a-z][a-z0-9-]{1,62}$`:

- starts with a lowercase letter
- 2-63 chars total
- only lowercase letters, digits, and hyphens

`metadata.version` must match `^\d+\.\d+\.\d+$` (strict semver, no prefix).

## Using these fixtures

```bash
# point an org's default-repo at a specific scenario
curl -X PUT https://sherpa/.../organizations/$ORG_ID/default-repo \
  -d '{"repo_url":"https://github.com/nadeem7sg/default","branch":"tdf-missing-version","pat_token":"..."}'

curl -X POST https://sherpa/.../organizations/$ORG_ID/default-repo/process
```

For preflight failure scenarios (Case F — `branch_not_found`), point any
existing scenario at a non-existent branch like `does-not-exist` instead of
creating a separate fixture branch.

## HITL / manual-retry / shadow-service breach

The repo content is the same as the failure branches above; what differs is
what you do *after* the run fails:

- **Manual retry of a failed validator**: run any `tdf-missing-*` or
  `dsdf-missing-*` branch, then `POST /runs/{run_id}/manual-retry` with
  `target_node_ids:["default_repo_tdf_schema_validator"]`.
- **Manual retry from preflight failure**: register `main` with
  `branch: does-not-exist`, then retry with
  `state_overrides:{"branch":"main"}`.
- **HITL upload-fix**: any rejected-file branch +
  `POST /runs/{run_id}/upload-fix` with corrected YAML.
- **SLA breach**: set `dag_policy.sla.nodes.default_repo_intake.max_duration_seconds: 1`
  and run against any branch.
- **Bias breach**: enable `dag_policy.bias_policy.enabled:true,
  escalate_to_llm:true`. Any branch works.
- **Model drift**: shadow service runs continuously; no fixture content
  required.
