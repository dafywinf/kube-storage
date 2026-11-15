# 🚀 kube-storage

`kube-storage` is a Helm umbrella chart that deploys and tests two Kubernetes storage
solutions: [Longhorn](https://longhorn.io)
and [local-path-provisioner](https://github.com/rancher/local-path-provisioner). It uses `fio` to benchmark persistent
volumes with configurable parameters.

## 📦 What's Included

- **Longhorn** – Distributed block storage
- **Local Path Storage** – Lightweight local volume provisioner
- **FIO Performance Test** – One Kubernetes Job per storage backend, **disabled by default** (see below)

## 🛠️ Deployment (Argo CD)

This chart is deployed by **Argo CD**, not by hand: `platform → storage → kube-storage` in
[homelab-gitops](https://github.com/dafywinf-homelab-platform/homelab-gitops), tracking `main` of this repo with
release name `kube-storage`, namespace `kube-storage` and `values.yaml`, plus
`longhorn.preUpgradeChecker.jobEnabled: false` as Longhorn recommends for Argo CD. **Merging to `main` deploys.** A
manual `helm upgrade` would be reverted.

Longhorn is pinned to an exact version in `Chart.yaml` and only supports upgrading **one minor version at a time**
(e.g. 1.11.x → 1.12.x). To preview locally:

```bash
helm dependency update
helm template kube-storage . -n kube-storage -f values.yaml
```

## 📊 Viewing FIO Performance Results

The benchmarks are **off by default**. Each Job stamps the current time into its pod template, and a Job's template
is immutable, so leaving them on breaks every later upgrade. They are also Helm hooks, so they are never cleaned up
automatically. To run one, set e.g. `longhorn-fio-performance-test.enabled: true` in `values.yaml` and merge. Once you
have read the results, set it back to `false` and delete the Job (`kubectl -n kube-storage delete job <name>`).

FIO logs are captured when the Job completes. To view the performance results:

```bash
kubectl get pods -n kube-storage

kubectl logs <fio-pod-name> -n kube-storage
```

Example:

```bash
kubectl logs job/local-path-fio-perf-job -n kube-storage
```

Look for key metrics in the log output:

- **IOPS**: e.g. `IOPS=15000`
- **Bandwidth**: e.g. `BW=60MiB/s`
- **Latency**: `avg=200us`, `99.9th=3ms`

## 🔁 Controlling Job Execution

To avoid concurrent job runs (which can cause disk contention), this chart uses Helm hooks to control execution order.
Each FIO test is implemented as a Helm-managed Job with:

* `helm.sh/hook: post-install,post-upgrade` annotations

This ensures jobs for different storage types (e.g., Longhorn, local-path) run in series rather than concurrently,
without the need for manual `sleep` delays.

You no longer need to manually delay execution using `sleep`.

## 🧼 Uninstalling

Remove the `kube-storage` Application from homelab-gitops. Its Application has no cascade-delete finalizer, so this
leaves the resources running: delete them afterwards if you really mean to remove Longhorn.

To uninstall Longhorn **safely**:

```bash
kubectl -n kube-storage patch settings.longhorn.io deleting-confirmation-flag \
  --type merge -p '{"value": "true"}'
```

Or manually:

```bash
kubectl -n kube-storage edit settings.longhorn.io deleting-confirmation-flag
# then set value: "true"
```
