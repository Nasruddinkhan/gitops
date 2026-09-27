# GitOps Manifests

This repository contains the OpenShift/Kubernetes YAML used to deploy:

- Oracle database
- `setting-service`

It supports four environments:

| Environment | Namespace |
| --- | --- |
| dev | `nasruddinkhan786-dev` |
| sit | `nasruddin-khan-sit` |
| uat | `nasruddin-khan-uat` |
| prod | `nasruddin-khan` |

## Repository Structure

```text
.
├── apps
│   ├── kustomization.yaml
│   ├── dev
│   │   ├── kustomization.yaml
│   │   ├── oracle.yaml
│   │   └── setting-service.yaml
│   ├── sit
│   │   ├── kustomization.yaml
│   │   ├── oracle.yaml
│   │   └── setting-service.yaml
│   ├── uat
│   │   ├── kustomization.yaml
│   │   ├── oracle.yaml
│   │   └── setting-service.yaml
│   └── prod
│       ├── kustomization.yaml
│       ├── oracle.yaml
│       └── setting-service.yaml
├── environments
│   ├── dev
│   ├── sit
│   ├── uat
│   └── prod
└── docs
    ├── gitops-guide.md
    └── openshift-runbook.md
```

## How Apps Are Activated

The root app selector is:

```text
apps/kustomization.yaml
```

Only one environment should be active there at a time.

Current active environment:

```yaml
resources:
  - dev
#  - sit
#  - uat
#  - prod
```

To activate SIT, change it to:

```yaml
resources:
#  - dev
  - sit
#  - uat
#  - prod
```

## What Each Folder Does

`apps/<env>/` contains Argo CD `Application` resources for one environment.

`environments/<env>/oracle/` contains the Oracle database manifests for one environment.

`environments/<env>/setting-service/` contains the Spring Boot service manifests for one environment.

`docs/` explains how this GitOps setup works and how to deploy it.

## Values You Must Replace

The Argo CD app files already point to this GitOps repository:

```yaml
repoURL: https://github.com/Nasruddinkhan/gitops.git
```

You still need to update the service image in each environment:

```text
environments/dev/setting-service/deployment.yaml
environments/sit/setting-service/deployment.yaml
environments/uat/setting-service/deployment.yaml
environments/prod/setting-service/deployment.yaml
```

Example Docker Hub image:

```yaml
image: docker.io/nasruddinkhan/setting-service:1.0.0
```

## Deployment Flow

The normal GitOps flow is:

1. Build and push the `setting-service` Docker image.
2. Update the image tag in the correct environment folder.
3. Activate the matching folder in `apps/kustomization.yaml`.
4. Commit and push this GitOps repo.
5. Argo CD syncs the active app into the matching OpenShift namespace.

Read the full explanation in:

```text
docs/gitops-guide.md
```
