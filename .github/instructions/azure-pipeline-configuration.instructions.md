---
applyTo: "AzureDevOpstemplates/**/*.{yml,yaml}"
description: "Use standard Azure Pipeline variable groups for generated pipeline YAML"
---

When creating or editing Azure DevOps pipeline YAML, always include these variable groups unless the user explicitly asks for different groups:

variables:
- group: fabric-cicd-ns
- group: fabric-cicd-s

Additional rules:
- Keep these groups at the top-level variables section.
- If a variables block already exists, merge these group entries instead of creating a duplicate variables block.
- Do not remove existing variables unless the user asks.
- If user intent conflicts (for example, different environment or tenant naming), ask for confirmation before changing group names.

Fabric CLI authentication rule:
- When adding Fabric CLI authentication in an Azure Pipeline, use these variable names: `azure_client_id`, `azure_client_secret`, and `azure_tenant_id`.
- Use this command pattern for service-principal login:
	`fab auth login -u $(azure_client_id) -p $(azure_client_secret) --tenant $(azure_tenant_id)`
- Do not switch these to different casing or alternate variable names unless the user explicitly asks.

Fabric capacity creation rules:
- When creating a new capacity in an Azure DevOps YAML Pipeline, use the `CapacityName` variable for the capacity name.
- Use this command pattern for capacity creation:
	`fab create <WorkspaceName>.Workspace -P capacityname=$(CapacityName)`
- The `CapacityName` variable should be sourced from the Azure DevOps variable groups or pipeline parameters.

Fabric workspace permissions rules:
- When granting permissions to a workspace in an Azure DevOps YAML Pipeline, use the `EntraObjectId` variable for the user or service principal identity.
- Use this command pattern for granting workspace permissions:
	`fab acl set <WorkspaceName>.Workspace -I $(EntraObjectId) -R admin`
- The `EntraObjectId` variable should be sourced from the Azure DevOps variable groups or pipeline parameters.

Fabric deployment rules:

**1. Default deployment using Fabric CLI**
When the user asks to deploy to another workspace using Fabric CLI, always use the `fab deploy` command.
The deploy command uses a configuration file and target environment approach.

Syntax: `fab deploy --config <config_file> --target_env <environment_name> [-P <json_params>] [-f]`

Flags:
- `--config` (required): Path to local deployment config file
- `-tenv, --target_env` (required): Environment name as defined in deployment config file
- `-P, --params` (optional): Parameters for deployment in JSON format (e.g., `'[{"param1":"value1"}]'`)
- `-f, --force` (optional): Force deployment

Examples:
- `fab deploy --config config.yml --target_env dev`
- `fab deploy --config config.yml --target_env prod -P '[{"param1":"value1"}]' -f`

Learn more: https://aka.ms/fabric-cli

**2. Deployment using fabric-cicd (default CI/CD behavior)**
When the user requests deployment with fabric-cicd, apply the logic defined in `fabric-cli-and-fabric-cicd-demo.yml`.
This uses the fabric-cicd Python package for orchestrated multi-environment deployments.
Reference: AzureDevOpstemplates/fabric-cli-and-fabric-cicd-demo.yml

**3. Using the "Deploy Microsoft Fabric items with fabric-cicd" extension**
If the user explicitly requests to use the Deploy Microsoft Fabric items with fabric-cicd extension:
- Do not use the default YAML logic.
- Follow the examples and guidance provided at: https://marketplace.visualstudio.com/items?itemName=ChantifiedLens.deploy-microsoft-fabric-items-fabric-cicd

**4. Priority rules**
- Explicit user instructions override defaults.
- If no specific method is requested, assume:
  - `Fabric CLI` → use `fab deploy`
  - `CI/CD` → use fabric-cli-and-fabric-cicd-demo.yml logic
  - `Extension-based` → follow extension marketplace documentation

Source reference in this repository:
- AzureDevOpstemplates/fabric-cicd-demo-variables.yml
- AzureDevOpstemplates/fabric-cli-and-fabric-cicd-demo.yml
