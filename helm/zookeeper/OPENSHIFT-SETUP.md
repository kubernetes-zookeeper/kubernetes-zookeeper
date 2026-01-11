# OpenShift Setup Guide for ZooKeeper with Standard SCC

This guide shows how to deploy ZooKeeper on OpenShift using the standard `nonroot-v2` SCC.

**Note:** The `nonroot-v2` SCC allows non-root UIDs and maintains security restrictions (no privileged containers, no host access, etc.). However, it may restrict UIDs to OpenShift's allocated ranges. If your StorageClass requires a specific UID outside the allocated range, you may need to use `anyuid` instead.

## Quick Start

### Step 1: Create Service Account

```bash
oc create serviceaccount zookeeper -n my-zk
```

### Step 2: Grant `nonroot-v2` SCC to Service Account

```bash
oc adm policy add-scc-to-user nonroot-v2 -z zookeeper -n my-zk
```

### Step 3: Deploy with Helm

```bash
helm upgrade --install my-zk ./helm/zookeeper --namespace my-zk --create-namespace \
  --values ./helm/zookeeper/values.yaml \
  --set AntiAffinity=hard \
  --set Cpu=2 \
  --set Servers=3 \
  --set Storage="250Gi" \
  --set StorageClass="zookeeper" \
  --set OpenShift=true \
  --set RunAsUser=1010 \
  --set FsGroup=1010 \
  --set ServiceAccount.name=zookeeper
```

## Alternative: Let Helm Create the Service Account

If you prefer to let Helm create the service account:

```bash
# Step 1: Deploy with Helm (Helm will create the service account and RoleBinding)
helm upgrade --install my-zk ./helm/zookeeper --namespace my-zk --create-namespace \
  --values ./helm/zookeeper/values.yaml \
  --set AntiAffinity=hard \
  --set Cpu=2 \
  --set Servers=3 \
  --set Storage="250Gi" \
  --set StorageClass="zookeeper" \
  --set OpenShift=true \
  --set RunAsUser=1010 \
  --set FsGroup=1010 \
  --set ServiceAccount.create=true \
  --set ServiceAccount.name=zookeeper
```

**Note:** When `OpenShift: true` is set, Helm automatically creates a RoleBinding to grant the `nonroot-v2` SCC to the ServiceAccount, so manual SCC assignment is not required.

## Verification

### Check SCC exists (nonroot-v2 is a standard SCC):
```bash
oc get scc nonroot-v2
```

### Check service account has SCC:
```bash
oc get serviceaccount zookeeper -n my-zk -o yaml
```

### Verify who can use the nonroot-v2 SCC:
```bash
oc adm policy who-can use scc nonroot-v2
```

### Verify pod security context:
```bash
oc get pod zk-my-zk-0 -n my-zk -o jsonpath='{.spec.securityContext}' | jq
```

Expected output:
```json
{
  "fsGroup": 1010,
  "runAsUser": 1010
}
```

### Verify pod can write to volume:
```bash
oc exec zk-my-zk-0 -n my-zk -- ls -la /var/lib/zookeeper
```

### Check pod is running:
```bash
oc get pods -n my-zk -l component=zk-my-zk
```

## Troubleshooting

### Error: "unable to validate against any security context constraint"

**Solution:** Verify the service account has the `nonroot-v2` SCC:
```bash
oc adm policy who-can use scc nonroot-v2
oc get serviceaccount zookeeper -n my-zk -o yaml
```

### Error: "forbidden: not usable by user or serviceaccount"

**Solution:** Grant the `nonroot-v2` SCC to the service account:
```bash
oc adm policy add-scc-to-user nonroot-v2 -z zookeeper -n my-zk
```

### Permission denied on volumes

**Solution:** Verify UID/GID match:
1. Check StorageClass mountOptions:
   ```bash
   oc get sc zookeeper -o yaml | grep -A 5 mountOptions
   ```

2. Check pod security context:
   ```bash
   oc get pod zk-my-zk-0 -n my-zk -o jsonpath='{.spec.securityContext}'
   ```

3. Both should show UID/GID 1010

## Cleanup

To remove everything:

```bash
# Delete Helm release
helm uninstall my-zk -n my-zk

# Remove nonroot-v2 SCC from service account (if granted manually)
oc adm policy remove-scc-from-user nonroot-v2 -z zookeeper -n my-zk

# Delete service account (if created manually)
oc delete serviceaccount zookeeper -n my-zk
```

**Note:** The `nonroot-v2` SCC is a standard OpenShift SCC and should not be deleted.

