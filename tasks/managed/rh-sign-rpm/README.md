# rh-sign-rpm Task

Tekton task to create internal requests to sign RPMs.

## Overview

This task processes RPM signing requests based on the artifacts.json file, which contains RPM metadata including architecture information. The task:

1. Reads the artifacts.json file to determine which RPM repositories need signing
2. Handles noarch RPMs specially (they are uploaded to all architecture repos)
3. Creates filtered artifacts.json files per architecture
4. Submits internal signing requests for each repository

## artifacts.json File

The `artifacts.json` file in this directory serves as an **example/reference** for the expected format.

### File Location at Runtime

During task execution, the artifacts.json file is read from:
```
$(params.dataDir)/$(params.artifactsJsonPath)
```

Default location: `/var/workdir/release/artifacts.json`

The file is typically populated by:
- The `use-trusted-artifact` step (from trusted artifacts)
- Previous pipeline steps that generate RPM metadata

### Format

The artifacts.json follows the pulp-tool format with RPM metadata:

```json
{
  "artifacts": {
    "package-name.rpm": {
      "labels": {
        "arch": "noarch|x86_64|aarch64|ppc64le|s390x|src",
        "date": "YYYY-MM-DD HH:MM:SS",
        "build_id": "build-identifier",
        "namespace": "namespace-name"
      },
      "url": "https://...",
      "sha256": "hash"
    }
  },
  "distributions": {
    "rpms": "https://...",
    "logs": "https://...",
    "sbom": "https://...",
    "artifacts": "https://..."
  }
}
```

### Architecture Handling

- **noarch**: RPMs are included in all architecture-specific repos (x86_64, aarch64, etc.)
- **src/source**: Source RPMs are processed separately
- **Architecture-specific**: Each arch gets its own filtered artifacts.json

When noarch RPMs are present:
1. The task filters out noarch from creating a separate repo
2. Each arch repo includes both arch-specific AND noarch RPMs
3. Distribution keys are used to find all repos with content

## Parameters

Key parameters:

- `artifactsJsonPath`: Relative path to artifacts.json within dataDir (default: "artifacts.json")
- `snapshotBuildId`: If provided, uses this as the repo name directly (bypasses dynamic detection)
- `dataDir`: Base directory where files are stored (default: "/var/workdir/release")

See the task spec for complete parameter documentation.

## Example Usage

When `snapshotBuildId` is not provided, the task automatically detects repositories from the artifacts.json architectures and distributions.

For the example artifacts.json in this directory (containing noarch and src RPMs):
- A "noarch" repo will NOT be created
- If distributions contain architecture keys (e.g., "x86_64", "aarch64"), each will get a signing request
- The "source" repo will be created for src RPMs
- Each arch repo's artifacts.json will include the noarch RPM

## Output

The task creates internal-request resources for the `direct-rpm-signing` pipeline with:
- Base64-encoded artifacts.json for each repository
- Kerberos authentication details extracted from DATA_FILE
- Service account and artifact storage configuration
- Fixed timeouts: 30m pipeline, 25m task
