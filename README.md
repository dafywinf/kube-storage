# LongHorn


## Deploying the Stack

```bash
helm dependency update  

helm upgrade --install kube-storage ./ \
  --namespace kube-storage \
  --create-namespace \
  --values values.yaml

# Confirm deployment successful
kubectl get all -n kube-storage
```


## (Optional) Uninstall Deployment

Here’s a concise and helpful addition you can include in your `README.md` under a section like **Uninstalling Longhorn**:

---

## 🧼 Uninstalling Longhorn

To safely uninstall Longhorn, you must first confirm deletion using the `deleting-confirmation-flag`. This prevents accidental data loss.

### 1. Set the Deletion Confirmation Flag

Before uninstalling, run the following to set the deletion flag to `true`:

```sh
kubectl -n kube-storage patch settings.longhorn.io deleting-confirmation-flag \
  --type merge -p '{"value": "true"}'
```

Alternatively, you can edit the setting manually:

```sh
kubectl -n kube-storage edit settings.longhorn.io deleting-confirmation-flag
```

Then change:

```yaml
value: "false"
```

to:

```yaml
value: "true"
```

### 2. Uninstall the Helm Release

Once the flag is set:

```sh
helm uninstall kube-storage --namespace kube-storage
```

### 3. (Optional) Clean Up Remaining Resources

To fully remove Longhorn components:

```sh
kubectl delete crds -l app=longhorn
kubectl delete namespace kube-storage
```
