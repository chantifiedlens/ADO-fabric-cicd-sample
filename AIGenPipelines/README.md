# GenWorkspace Fabric CLI Pipeline

This folder contains [create-genworkspace-fabric-cli.yml](create-genworkspace-fabric-cli.yml), an Azure DevOps pipeline that uses Microsoft Fabric CLI (fab) to:

1. Authenticate with a service principal.
2. Ensure the workspace GenWorkspace exists (create only when missing).
3. Grant Admin permissions to the identity in EntraObjectId.
4. Deploy Fabric items from the local workspace folder using:
   - config: workspace/config.yml
   - command: fab deploy --config <path>/workspace/config.yml --target_env <env> -f

## What The Pipeline Does

Pipeline file: [create-genworkspace-fabric-cli.yml](create-genworkspace-fabric-cli.yml)

High-level flow:

1. Installs Python and ms-fabric-cli.
2. Authenticates with:
   - fab auth login -u $(azure_client_id) -p $(azure_client_secret) --tenant $(azure_tenant_id)
3. Checks whether GenWorkspace exists with fab exists.
4. Creates GenWorkspace only if it does not exist:
   - fab create GenWorkspace.Workspace -P capacityname=$(CapacityName)
5. Assigns admin access:
   - fab acl set GenWorkspace.Workspace -I $(EntraObjectId) -R admin -f
6. Deploys items from [workspace/config.yml](../workspace/config.yml) to the target environment configured in pipeline variable TargetEnvironment (default: test):
   - fab deploy --config "$(Build.SourcesDirectory)/workspace/config.yml" --target_env "$(TargetEnvironment)" -f

## Create A New Azure DevOps Pipeline For This YAML

1. Open Azure DevOps and go to Pipelines.
2. Select New pipeline.
3. Choose your repo containing this folder.
4. Choose Existing Azure Pipelines YAML file.
5. Select [AIGenPipelines/create-genworkspace-fabric-cli.yml](create-genworkspace-fabric-cli.yml).
6. Continue and run the pipeline.
7. If prompted for permissions, authorize pipeline access to variable groups.

## Create Azure Pipeline Variable Groups

The YAML expects these variable groups at the top-level variables block:

1. fabric-cicd-ns
2. fabric-cicd-s

Create a variable group:

1. Azure DevOps Project Settings.
2. Pipelines -> Library.
3. New variable group.
4. Name it (for example fabric-cicd-ns).
5. Add variables (see required list below).
6. Enable pipeline access:
   - Option A: Check Allow access to all pipelines.
   - Option B: Keep restricted and explicitly authorize this pipeline when first run.
7. Save.
8. Repeat for the second group (fabric-cicd-s) if you separate variables by scope/environment.

## Required Variables For This Pipeline

Required by authentication, workspace creation, and ACL assignment:

1. azure_client_id
2. azure_client_secret
3. azure_tenant_id
4. CapacityName
5. EntraObjectId

Used by deploy behavior in the YAML file:

1. TargetEnvironment (pipeline variable, default is test)

Notes:

1. Keep azure_client_secret marked secret in Azure DevOps.
2. The target environment name must exist in [workspace/config.yml](../workspace/config.yml) under core.workspace (for example test, prod).
3. In this repo config, test maps to GenWorkspace.

## Fabric CLI Overview

Source: https://github.com/microsoft/fabric-cli

Microsoft Fabric CLI (fab) is a file-system-inspired command-line interface for Microsoft Fabric that supports automation and CI/CD usage.

Key points from the project README:

1. Install with pip:
   - pip install ms-fabric-cli
2. Python support:
   - 3.10, 3.11, 3.12, 3.13
3. Cross-platform:
   - Windows, macOS, Linux
4. Auth modes include:
   - Interactive login
   - Service principal (client secret/cert/federated patterns)
   - Managed identity
5. CLI supports both:
   - Command-line mode for automation
   - Interactive mode for exploration
6. Familiar command model:
   - ls, cd, mkdir, cp, rm, exists, get, api, job, deploy

Why it is used here:

1. Deterministic automation in Azure Pipelines.
2. Native workspace/item operations plus deployment via config-driven fab deploy.

## Troubleshooting

### 1. Authentication fails

Symptoms:

1. fab auth login returns unauthorized/invalid client/tenant.

Checks:

1. Verify azure_client_id, azure_client_secret, azure_tenant_id values in variable groups.
2. Confirm service principal has Microsoft Fabric permissions.
3. Confirm secret is valid and not expired.
4. Validate tenant ID matches the service principal tenant.

### 2. Variable group permission issues

Symptoms:

1. Pipeline cannot read variables or asks for authorization.

Checks:

1. Verify the variable group names are exactly fabric-cicd-ns and fabric-cicd-s.
2. Grant pipeline access to variable groups in Library permissions.
3. Re-run and authorize when Azure DevOps prompts.

### 3. Target environment mismatch in config.yml

Symptoms:

1. fab deploy fails with missing/unknown target environment.

Checks:

1. Confirm TargetEnvironment value (for example test/prod) exists in [workspace/config.yml](../workspace/config.yml).
2. Confirm the config path in the pipeline points to workspace/config.yml.
3. Ensure core.workspace contains the selected target environment key.

## MCP Servers Used

1. None.
2. This document was generated using workspace file reads and a direct webpage fetch of the GitHub repository page, not via an MCP server.
