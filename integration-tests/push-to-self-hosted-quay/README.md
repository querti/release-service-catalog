# push-to-self-hosted-quay test
## Overview
This test is a variant of `push-to-external-registry` that uses a **self-hosted Quay** instance
as both the build output (intermediate) registry and the release destination registry, instead of
quay.io.

## Setup
### Dependencies
* GitHub repo: https://github.com/hacbs-release-tests/e2e-base
* GitHub personal access token (classic) for above repo with **admin:repo_hook**, **delete_repo**, **repo** scopes.
* The password to the vault files. (Contact a member of the Release team should you want to run this
  test suite.)
* Access to the target cluster and tenant and managed namespaces
  * This test uses stg-rh01 and the dev-release-team-tenant and managed-release-team-tenant namespaces.
* A self-hosted Quay instance accessible from the build/managed clusters
* Push credentials (robot account or token) for the self-hosted Quay instance

### Required Environment Variables
- GITHUB_TOKEN
  - The GitHub personal access token needed for repo operations
  - The repo in question can be located in [test.env](test.env)
- VAULT_PASSWORD_FILE
  - This is the path to a file that contains the ansible vault
    password needed to decrypt the secrets needed for testing.
- RELEASE_CATALOG_GIT_URL
  - The release service catalog URL to use in the RPA
  - This is provided when testing PRs
- RELEASE_CATALOG_GIT_REVISION
  - The release service catalog revision to use in the RPA
  - This is provided when testing PRs

### Optional Environment Variables
- KUBECONFIG
  - The KUBECONFIG file to used to login to the target cluster
  - This is provided when testing PRs

### Test Properties
#### [test.env](test.env)
- This file contains resource names and configuration values needed for testing.
- `self_hosted_quay_url` - The URL of the self-hosted Quay instance (e.g. `quay-server.example.com`)
- `self_hosted_quay_org` - The organization on the self-hosted Quay instance

### Key Differences from push-to-external-registry
- **Build output**: Uses `spec.containerImage` pointing to self-hosted Quay instead of
  `image.redhat.com/generate` (which auto-creates repos on quay.io)
- **Release destination**: Pushes to self-hosted Quay instead of `quay.io/hacbs-release-tests/`
- **make-repo-public**: Skipped automatically (only supports quay.io API)
- **Secrets**: Credentials target the self-hosted Quay instance

### Test Functions
#### [lib/test-functions.sh](../lib/test-functions.sh)
- This file contains re-usable functions for tests

### Secrets
- Secrets needed for testing are stored in ansible vault files.
  - [vault/managed-secrets.yaml](vault/managed-secrets.yaml)
  - [vault/tenant-secrets.yaml](vault/tenant-secrets.yaml)
- Most secrets required are contained in the files above.

### Running the test

```shell
integration-tests/run-test.sh push-to-self-hosted-quay
```

### Debugging

There is a `--skip-cleanup` option to the script in the event that you want to examine the resources
after a test has ended.

### Maintenance
- Should you require to add or update a secret, follow these steps:
```shell
ansible-vault decrypt vault/tenant-secrets.yaml --output "/tmp/tenant-secrets.yaml" --vault-password-file <vault password file>
```

```shell
vi /tmp/tenant-secrets.yaml
```

```shell
ansible-vault encrypt /tmp/tenant-secrets.yaml --output "vault/tenant-secrets.yaml" --vault-password-file <vault password file>
```

```shell
rm /tmp/tenant-secrets.yaml
```
