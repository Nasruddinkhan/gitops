# OpenShift Runbook

This runbook shows how to apply and check the GitOps manifests.

## Namespaces

| Environment | Namespace |
| --- | --- |
| dev | `nasruddinkhan786-dev` |
| sit | `nasruddin-khan-sit` |
| uat | `nasruddin-khan-uat` |
| prod | `nasruddin-khan` |

Switch to one namespace:

```bash
oc project nasruddin-khan-sit
```

## Activate One Argo CD App Environment

Edit:

```text
apps/kustomization.yaml
```

Keep only one environment active.

Dev active:

```yaml
resources:
  - dev
#  - sit
#  - uat
#  - prod
```

Prod active:

```yaml
resources:
#  - dev
#  - sit
#  - uat
  - prod
```

Then apply the active app set:

```bash
oc apply -k apps
```

## Apply One Environment Directly

You can test an environment without Argo CD by using `oc apply -k`.

Apply dev:

```bash
oc apply -k environments/dev
```

Apply SIT:

```bash
oc apply -k environments/sit
```

Apply UAT:

```bash
oc apply -k environments/uat
```

Apply prod:

```bash
oc apply -k environments/prod
```

## Apply Only Oracle

```bash
oc apply -k environments/sit/oracle
```

Check Oracle:

```bash
oc get deployment oracle-db
oc get pods -l app=oracle-db
oc get pvc oracle-db-data
oc get svc oracle-db
```

View logs:

```bash
oc logs deployment/oracle-db
```

## Apply Only Setting Service

Before applying `setting-service`, replace the image placeholder in the environment you are deploying.

Examples:

```text
environments/sit/setting-service/deployment.yaml
environments/uat/setting-service/deployment.yaml
environments/prod/setting-service/deployment.yaml
```

Then apply one service environment:

```bash
oc apply -k environments/sit/setting-service
```

Check the deployment:

```bash
oc get deployment setting-service
oc get pods -l app=setting-service
oc get svc setting-service
oc get route setting-service
```

## Test Database Service DNS

From inside the same namespace, `setting-service` connects to Oracle with:

```text
jdbc:oracle:thin:@//oracle-db:1521/FREEPDB1
```

The service name is:

```text
oracle-db
```

The port is:

```text
1521
```

## Common Checks

Check all resources in the current namespace:

```bash
oc get all
```

Check Oracle:

```bash
oc get deployment oracle-db
oc get pods -l app=oracle-db
oc get svc oracle-db
```

Check setting service:

```bash
oc get deployment setting-service
oc get pods -l app=setting-service
oc get svc setting-service
oc get route setting-service
```

## Common Problems

If the Oracle pod is pending, check the PVC:

```bash
oc describe pvc oracle-db-data
```

If `setting-service` cannot connect to Oracle, check:

```bash
oc get svc oracle-db
oc logs deployment/setting-service
oc logs deployment/oracle-db
```

If Argo CD does not sync, check that `repoURL` points to the real Git repo and that Argo CD has permission to read it.
