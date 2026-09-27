# GitOps Guide

GitOps means the desired state of your infrastructure and applications is stored in Git.

Instead of manually creating deployments from the OpenShift UI every time, you keep YAML files in this repo. Argo CD watches the repo and applies those YAML files to the OpenShift cluster.

## Environments

This repo has four environments:

| Environment | Namespace | Purpose |
| --- | --- | --- |
| dev | `nasruddinkhan786-dev` | Development and first testing |
| sit | `nasruddin-khan-sit` | System integration testing |
| uat | `nasruddin-khan-uat` | User acceptance testing |
| prod | `nasruddin-khan` | Production |

Each environment has its own folder under `apps/` and `environments/`.

```text
apps/dev                 environments/dev
apps/sit                 environments/sit
apps/uat                 environments/uat
apps/prod                environments/prod
```

The `apps/<env>/` folder contains Argo CD `Application` YAML.

The `environments/<env>/` folder contains the actual Kubernetes/OpenShift workload YAML.

## One Active App Environment

The root app selector is:

```text
apps/kustomization.yaml
```

Only one app environment should be active at a time.

Example with dev active:

```yaml
resources:
  - dev
#  - sit
#  - uat
#  - prod
```

Example with prod active:

```yaml
resources:
#  - dev
#  - sit
#  - uat
  - prod
```

This keeps the app entrypoint simple and avoids accidentally creating Argo CD apps for all environments at once.

## How Argo CD Reads This Repo

Each environment folder under `apps/` contains two Argo CD apps:

```text
oracle.yaml
setting-service.yaml
```

Example for SIT:

```text
apps/sit/oracle.yaml
apps/sit/setting-service.yaml
```

These point to:

```text
environments/sit/oracle
environments/sit/setting-service
```

and deploy into:

```text
nasruddin-khan-sit
```

## Oracle Database

Oracle is defined in each environment under:

```text
environments/<env>/oracle/oracle-deployment.yaml
```

It creates:

- A `Secret` named `oracle-db-secret`
- A `PersistentVolumeClaim` named `oracle-db-data`
- A `Deployment` named `oracle-db`
- A `Service` named `oracle-db`

Other pods in the same namespace connect to Oracle using:

```text
oracle-db
```

The JDBC URL used by `setting-service` is:

```text
jdbc:oracle:thin:@//oracle-db:1521/FREEPDB1
```

## Setting Service

The Spring Boot app is defined in each environment under:

```text
environments/<env>/setting-service/deployment.yaml
```

It creates:

- A `Deployment` named `setting-service`
- A `Service` named `setting-service`
- An OpenShift `Route` named `setting-service`

The app connects to Oracle through:

```yaml
SPRING_DATASOURCE_URL
SPRING_DATASOURCE_USERNAME
SPRING_DATASOURCE_PASSWORD
```

The username and password are read from:

```text
oracle-db-secret
```

## Promotion Flow

A common GitOps promotion flow looks like this:

1. Build and test a new image in `dev`.
2. Put the same image tag into `sit`.
3. After SIT passes, put the same image tag into `uat`.
4. After UAT sign-off, put the same image tag into `prod`.

Example Docker Hub image:

```yaml
image: docker.io/nasruddinkhan/setting-service:1.0.0
```

## Important Notes

Do not expose Oracle using an OpenShift `Route`. Routes are for HTTP/HTTPS traffic. Oracle uses TCP on port `1521`, so it should stay behind a Kubernetes `Service`.

Do not commit real production passwords into Git. The current secret values are starter values for development. For serious use, use Sealed Secrets, External Secrets, or manually created OpenShift secrets.

For databases, a `StatefulSet` is usually better than a `Deployment`. This repo uses a `Deployment` because that is what you asked for, and it is acceptable for learning or simple non-critical environments.
