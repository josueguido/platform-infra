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

## Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/) so commit
history is consistent and easy to scan. Use this format:

```text
<type>(<scope>): <short description>
```

The scope is optional. Use an imperative, concise description. Common types:

| Type | Use |
| --- | --- |
| `feat` | Add a capability or infrastructure component |
| `fix` | Correct a defect or unintended behavior |
| `docs` | Change documentation |
| `refactor` | Restructure code without changing behavior |
| `test` | Add or update tests |
| `ci` | Change CI/CD configuration |
| `chore` | Make maintenance changes |

Examples:

```text
feat(networking): add private subnets
fix(eks): correct node group capacity settings
docs: describe the branch workflow
```

For an incompatible change, add `!` after the type or scope and describe the
breaking change in the commit body or footer:

```text
feat(eks)!: replace the legacy cluster input
```

## Branch workflow

Keep `main` as the stable integration branch. Use short-lived branches for
changes; do not create permanent branches for `dev`, `staging`, or
`production`. Those environments are represented by directories under
`environments/`.

Use a type-prefixed branch name in lowercase kebab-case:

```text
feat/add-eks-module
fix/private-subnet-routes
docs/branch-workflow
chore/update-terraform-version
```

Typical workflow:

1. Update your local `main` branch and create a branch for one focused change.
2. Make and validate the change; use Conventional Commit messages.
3. Push the branch and open a pull request targeting `main`.
4. Review the Terraform plan and obtain the required approvals before merging,
   especially for changes that affect production.
5. Merge the pull request and delete the short-lived branch.

Example:

```sh
git switch main
git pull
git switch -c feat/add-eks-module
# Make changes and commit them
git push -u origin feat/add-eks-module
```
