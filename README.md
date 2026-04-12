# Azure DevOps + Fabric CI/CD sample

This repository shows how to operationalize Microsoft Fabric deployments from Azure DevOps YAML pipelines using:

- [`fabric-cicd`](https://github.com/microsoft/fabric-cicd)
- [`ms-fabric-cli` (`fab`)](https://github.com/microsoft/fabric-cli)

If you want to test locally first, use [`auth_local_hard_coded.py`](./auth_local_hard_coded.py) (it includes starter comments).

## Pipeline scenarios covered in this repo

This repo contains **five** YAML pipeline scenarios:

1. [`AzureDevOpstemplates/fabric-cicd-demo-variables.yml`](./AzureDevOpstemplates/fabric-cicd-demo-variables.yml)  
	Uses Azure DevOps variable groups only (good when values are mostly fixed).
2. [`AzureDevOpstemplates/fabric-cicd-demo-wsparameters.yml`](./AzureDevOpstemplates/fabric-cicd-demo-wsparameters.yml)  
	Uses pipeline parameters for workspace-dependent values (workspace IDs, items in scope, resource URL).
3. [`AzureDevOpstemplates/fabric-cicd-demo-gblparameters.yml`](./AzureDevOpstemplates/fabric-cicd-demo-gblparameters.yml)  
	Uses global pipeline parameters (including credentials and workspace values).
4. [`AzureDevOpstemplates/fabric-cli-and-fabric-cicd-demo.yml`](./AzureDevOpstemplates/fabric-cli-and-fabric-cicd-demo.yml)  
	Creates a workspace with `fab`, grants ACL, then deploys content with `fabric-cicd`.
5. [`AIGenPipelines/create-genworkspace-fabric-cli.yml`](./AIGenPipelines/create-genworkspace-fabric-cli.yml)  
	Uses `fab` end-to-end to ensure/create workspace, set permissions, and deploy using **`fab deploy`**.

## YAML scenario using `fab deploy`

The pipeline [`AIGenPipelines/create-genworkspace-fabric-cli.yml`](./AIGenPipelines/create-genworkspace-fabric-cli.yml) covers the **configuration deployment** pattern with:

- `fab auth login` for SPN auth
- `fab exists` + `fab create` to ensure the workspace exists
- `fab acl set` to grant admin access
- `fab deploy --config <config-file> --target_env <environment> -f` to deploy content

Reference documentation for this deployment style:  
https://microsoft.github.io/fabric-cicd/0.3.1/how_to/config_deployment/

> Note: This pipeline points to `workspace/config.yml`. If you do not have that file yet, add it first and ensure your `target_env` (for example `Test`) exists in the config mappings.

## Required variables and parameters for each YAML pipeline

### 1) `fabric-cicd-demo-variables.yml`

**Parameters required:** none  
**Variables required (via variable groups):**

- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_TENANT_ID`
- `resourceUrl` (typically `https://api.fabric.microsoft.com`)
- `TestWorkspace` (GUID)
- `ProdWorkspace` (GUID)
- `ItemsInScope` (for example `Notebook,Environment,Report,SemanticModel,MirroredDatabase`)

**Variable groups referenced in YAML:**

- `fabric-cicd-ns`
- `fabric-cicd-s`

### 2) `fabric-cicd-demo-wsparameters.yml`

**Parameters required:**

- `ItemsInScope`
- `resourceUrl`
- `ProdWorkspace`
- `TestWorkspace`

**Variables required (via variable group):**

- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_TENANT_ID`

**Variable groups referenced in YAML:**

- `fabric-cicd-s`

### 3) `fabric-cicd-demo-gblparameters.yml`

**Parameters required:**

- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_TENANT_ID`
- `ItemsInScope`
- `resourceUrl`
- `ProdWorkspace`
- `TestWorkspace`

**Variables required:** none (all key inputs are parameters)

### 4) `fabric-cli-and-fabric-cicd-demo.yml`

**Parameters required:**

- `WorkspaceName`

**Variables required (via variable groups):**

- `azure_client_id`
- `azure_client_secret`
- `azure_tenant_id`
- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_TENANT_ID`
- `CapacityName`
- `EntraObjectId`
- `resourceUrl`
- `ItemsInScope`

**Variable groups referenced in YAML:**

- `fabric-cicd-ns`
- `fabric-cicd-s`

### 5) `create-genworkspace-fabric-cli.yml`

**Parameters required:** none

**Pipeline variables defined in YAML:**

- `WorkspaceName` (default: `GenWorkspace`)
- `TargetEnvironment` (default: `Test`)

**Variables required (via variable groups):**

- `azure_client_id`
- `azure_client_secret`
- `azure_tenant_id`
- `CapacityName`
- `EntraObjectId`

**Variable groups referenced in YAML:**

- `fabric-cicd-ns`
- `fabric-cicd-s`

## Quick start

1. [Import this repository into Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/repos/git/import-git-repository?view=azure-devops&WT.mc_id=DP-MVP-5004032%3Fview%3Dazure-devops)
2. Create a pipeline from one of the YAML files above.
3. Configure variable groups/parameters based on the selected pipeline.
4. Run `Test` stage first, then promote to `Prod`.

Related write-up:  
[Operationalize fabric-cicd to work with Microsoft Fabric and YAML Pipelines](https://ChantifiedLens.com/2025/03/18/operationalize-fabric-cicd-to-work-with-microsoft-fabric-and-yaml-pipelines/)

## Workspace sample content

Most sample Fabric items under [`workspace/`](./workspace/) are based on examples from the upstream [`microsoft/fabric-cicd`](https://github.com/microsoft/fabric-cicd) project.

You can customize [`workspace/parameter.yml`](./workspace/parameter.yml) to fit your environment.

## What the `.github` subfolder contains

The `.github` folder in this repo contains Copilot/agent guidance and reusable skills.

### `.github/copilot-instructions.md`

Repository-level Copilot instructions for Fabric CLI conventions, path model, auth patterns, and safe usage.

### `.github/agents/`

- [`fab.agent.md`](./.github/agents/fab.agent.md): a dedicated chat agent profile focused on operating Microsoft Fabric through `fab` commands.

### `.github/instructions/`

- [`azure-pipeline-configuration.instructions.md`](./.github/instructions/azure-pipeline-configuration.instructions.md): policy guidance for Azure DevOps YAML in this repo, including:
  - expected variable groups (`fabric-cicd-ns`, `fabric-cicd-s`)
  - Fabric CLI auth variable names (`azure_client_id`, `azure_client_secret`, `azure_tenant_id`)
  - default deployment guidance (`fab deploy` vs `fabric-cicd` orchestration)

### `.github/skills/`

- [`README.md`](./.github/skills/README.md): overview of available skills and when to use them.
- [`fabric-cli-core/`](./.github/skills/fabric-cli-core/): core Fabric CLI skill pack
  - [`SKILL.md`](./.github/skills/fabric-cli-core/SKILL.md)
  - `references/` docs (quickstart, full command reference, notebooks, workspaces, reports, semantic models, APIs)
  - `scripts/health_check.py`
- [`fabric-cli-powerbi/`](./.github/skills/fabric-cli-powerbi/): Power BI-focused Fabric CLI skill pack
  - [`SKILL.md`](./.github/skills/fabric-cli-powerbi/SKILL.md)
  - `references/` docs (refresh, DAX, reports, gateways)
  - scripts: `refresh_model.py`, `rebind_report.py`, `list_refresh_history.py`

## License

This repository is provided "as is" under the [MIT License](https://opensource.org/licenses/MIT).
