# platform-infra

Terraform infrastructure-as-code repository. The goal is to keep reusable
components in `modules/` and compose them from `environments/`, allowing
different projects to share the same infrastructure foundation.

> This repository currently contains only the initial scaffold. It does not
> provision resources or configure providers, backends, or credentials.

## Structure

```text
platform-infra/
├── modules/
│   ├── networking/
│   ├── eks/
│   ├── iam/
│   ├── ecr/
│   ├── security-groups/
│   └── monitoring/
└── environments/
    ├── dev/
    │   └── main.tf
    ├── staging/
    │   └── main.tf
    └── production/
        └── main.tf
```

### Modules

Each module should encapsulate one infrastructure responsibility and be
configurable through explicit inputs. When implementing a module, keep its
resources in `main.tf`, its inputs in `variables.tf`, and its outputs in
`outputs.tf`. Avoid including project- or environment-specific values in a
module.

| Module | Intended responsibility |
| --- | --- |
| `networking` | VPC, subnets, and shared networking components |
| `eks` | EKS cluster and related baseline configuration |
| `iam` | Reusable IAM roles and policies |
| `ecr` | Container image repositories |
| `security-groups` | Security groups and traffic rules |
| `monitoring` | Monitoring components and configuration |

### Environments

Each directory in `environments/` represents an independent configuration
that composes the required modules. Keep environment-specific differences in
their variables and configuration rather than duplicating module
implementations.

Before adding backends, providers, or environment values, decide how remote
state, AWS access, and secrets will be managed. Do not store credentials or
secrets in the repository or in version-controlled `.tfvars` files.

## Clusters and applications

You do not need to create an EKS cluster for every application. Multiple
applications, such as TaskFlow and Inventory API, can be deployed in separate
namespaces within a shared cluster when isolation, permissions, and capacity
requirements allow. A namespace alone is not always a sufficient security
boundary; add access controls, network policies, and resource limits as
required by the project.

## Getting started

From the repository root, once an environment has Terraform configuration:

```sh
terraform -chdir=environments/dev init
terraform -chdir=environments/dev plan
```

Review the plan before applying changes. Configure state and credentials
according to the team's policies before provisioning infrastructure.
