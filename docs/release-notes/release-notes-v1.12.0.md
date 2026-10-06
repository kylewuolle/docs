# k0rdent 1.12.0 Release Notes

**Release date:** October 6, 2026

## Components Versions

| Provider Name                        | Version          |
|--------------------------------------|------------------|
| Cluster API                          | v1.14.2          |
| Cluster API Provider AWS             | v2.13.0          |
| Cluster API Provider Azure           | v1.26.0          |
| Cluster API Provider Docker          | v1.14.2          |
| Cluster API Provider GCP             | v1.13.1          |
| Cluster API Provider Infoblox        | v0.2.2           |
| Cluster API Provider IPAM            | v1.1.0           |
| Cluster API Provider k0smotron       | v2.1.1           |
| Cluster API Provider Kubevirt        | v0.11.2          |
| Cluster API Provider OpenStack (ORC) | v0.14.7 (v2.1.0) |
| Cluster API Provider vSphere         | v1.16.1          |
| Projectsveltos                       | v1.14.0          |
| k0s (control plane runtime)          | v1.36.3          |
| cert-manager (charts)                | v1.21.2          |
| CAPI Operator                        | v0.29.0          |

---

## Highlights

* **Child Cluster RBAC Policy**: The new `RBACPolicy` resource defines a catalog of `ClusterRole` to user and group
bindings. The RBAC operator applies it to the child cluster of every `ClusterDeployment` that references it via
`spec.rbacPolicy`, prunes bindings that are no longer desired, re-syncs periodically to correct drift, and reports
the result in the new `RBACPolicyReady` condition.

* **Access Management Redesign**: `AccessManagement` access rules now use a generic `resources` list that can
distribute any namespaced Kind, including custom resources, selected by names or label selectors. The `RBACPolicy`
resource is supported as a built-in Kind. Per-Kind distribution errors are reported in the `AccessManagement` status.

* **Accurate Service Deployment State**: The `ServiceSet` controller now tracks what is actually deployed by hashing
the release, values and patches, and verifies the health of deployed workloads (Deployments, StatefulSets,
DaemonSets, Pods, PVs and PVCs) before marking services as `Deployed`.

* **Safer Service Upgrades**: `ServiceTemplateChain` upgrades now walk every intermediate version of a multi-hop
upgrade path, a failed Helm upgrade no longer causes the deployed release to be deleted and reinstalled,
and a `MultiClusterService` waiting on another `MultiClusterService` now reports it in its status.

* **Reduced Load on Regional and Child Clusters**: KCM now caches one RESTMapper per target API server instead of
running API discovery for every client, which significantly reduces discovery requests. The cache size can be tuned
with `controller.restMapperCacheMaxEntries` in the KCM chart.

* **Platform Updates**: Cluster API v1.14.2, k0smotron v2.1.1, Projectsveltos v1.14.0, k0s v1.36.3+k0s.2,
and etcd v3.7.1 for hosted control plane templates.

## Upgrade Notes

### AccessManagement Deprecated Fields

The `clusterTemplateChains`, `serviceTemplateChains`, `credentials`, `clusterAuthentications`, `dataSources` and
`clusterAuditPolicies` fields of `AccessManagement` access rules are deprecated in favor of the generic `resources`
list. Existing configurations keep working: the deprecated fields are automatically migrated to equivalent
`resources` entries on write. Update your manifests to use `resources`, because the deprecated fields will be
removed in a future API version.

### ServiceSet Deployment State

`ServiceSet` objects now transition to the `Deployed` state only after the health checks of the deployed workloads
pass. Services whose workloads are not healthy remain in their previous state, even if the Helm release was applied.

---

## Changelog

### New Features

* **feat**: RBAC policy operator ([#3045](https://github.com/k0rdent/kcm/pull/3045)) by @eromanova
* **feat**: access management redesign ([#3010](https://github.com/k0rdent/kcm/pull/3010)) by @eromanova
* **feat**: show creation of ServiceSet by MCS blocked by another MCS in status ([#2910](https://github.com/k0rdent/kcm/pull/2910)) by @wahabmk
* **feat(ksm)**: track deployment state ([#2891](https://github.com/k0rdent/kcm/pull/2891)) by @BROngineer

---

### Notable Fixes

* **fix(am)**: skip distribution into the same and system namespaces ([#3074](https://github.com/k0rdent/kcm/pull/3074)) by @zerospiel
* **fix**: RBAC policy operator follow-ups ([#3065](https://github.com/k0rdent/kcm/pull/3065)) by @eromanova
* **fix(ksm)**: ServiceTemplateChain upgrade skips intermediate versions ([#3046](https://github.com/k0rdent/kcm/pull/3046)) by @josef-hak
* **fix**: Add default rule for Job-owned pods ([#3028](https://github.com/k0rdent/kcm/pull/3028)) by @wahabmk
* **fix**: share one RESTMapper per cluster instead of rebuilding per client ([#3008](https://github.com/k0rdent/kcm/pull/3008)) by @srnbckr
* **fix**: doc comment for HelmAction field ([#2997](https://github.com/k0rdent/kcm/pull/2997)) by @wahabmk
* **fix**: unintended release deletion on failed upgrade ([#3001](https://github.com/k0rdent/kcm/pull/3001)) by @BROngineer
* **fix**: allow controller to delete configmaps ([#2989](https://github.com/k0rdent/kcm/pull/2989)) by @eromanova

---

### Platform & Dependency Updates

* **chore(deps)**: bump CAPI to v1.14.2, k0smotron to v2.1.1 and kcm-regional dependencies ([bd8d3e0](https://github.com/k0rdent/kcm/commit/bd8d3e008f560623c76a4bad574b27199875db38)) by @zerospiel
* **chore(deps)**: bump github.com/cert-manager/cert-manager ([#3070](https://github.com/k0rdent/kcm/pull/3070)) by @dependabot
* **chore(deps)**: bump golang.org/x/net from 0.58.0 to 0.59.0 ([#3069](https://github.com/k0rdent/kcm/pull/3069)) by @dependabot
* **chore(deps)**: bump github.com/onsi/ginkgo/v2 from 2.32.1 to 2.32.2 ([#3068](https://github.com/k0rdent/kcm/pull/3068)) by @dependabot
* **chore(deps)**: bump helm.sh/helm/v3 from 3.21.4 to 3.22.0 ([#3067](https://github.com/k0rdent/kcm/pull/3067)) by @dependabot
* **chore(deps)**: bump golang.org/x/text from 0.41.0 to 0.42.0 ([#3061](https://github.com/k0rdent/kcm/pull/3061)) by @dependabot
* **chore(deps)**: bump sigs.k8s.io/cluster-api from 1.14.1 to 1.14.2 ([#3062](https://github.com/k0rdent/kcm/pull/3062)) by @dependabot
* **chore(deps)**: bump sigs.k8s.io/cluster-api/api from 1.14.1 to 1.14.2 ([#3063](https://github.com/k0rdent/kcm/pull/3063)) by @dependabot
* **chore(deps)**: bump kubevirt.io/containerized-data-importer-api ([#3059](https://github.com/k0rdent/kcm/pull/3059)) by @dependabot
* **chore(deps)**: bump golang.org/x/time from 0.15.0 to 0.16.0 ([#3057](https://github.com/k0rdent/kcm/pull/3057)) by @dependabot
* **chore(deps)**: bump sigs.k8s.io/cluster-api from 1.14.0 to 1.14.1 ([#3054](https://github.com/k0rdent/kcm/pull/3054)) by @dependabot
* **chore(deps)**: bump golang.org/x/sync from 0.22.0 to 0.23.0 ([#3055](https://github.com/k0rdent/kcm/pull/3055)) by @dependabot
* **chore(deps)**: bump sigs.k8s.io/cluster-api/api from 1.14.0 to 1.14.1 ([#3056](https://github.com/k0rdent/kcm/pull/3056)) by @dependabot
* **chore(deps)**: bump projectsveltos to 1.14.0 ([#3029](https://github.com/k0rdent/kcm/pull/3029)) by @BROngineer
* **chore**: bump etcd from 3.5.13 to 3.7.1 ([#3051](https://github.com/k0rdent/kcm/pull/3051)) by @zerospiel
* **chore(deps)**: bump github.com/fluxcd/pkg/runtime from 0.111.0 to 0.112.0 ([#3053](https://github.com/k0rdent/kcm/pull/3053)) by @dependabot
* **chore(deps)**: bump golang.org/x/crypto from 0.55.0 to 0.56.0 ([#3052](https://github.com/k0rdent/kcm/pull/3052)) by @dependabot
* **chore(deps)**: bump github.com/fluxcd/source-controller/api ([#3050](https://github.com/k0rdent/kcm/pull/3050)) by @dependabot
* **chore(deps)**: bump k8s.io/apiserver from 0.36.4 to 0.37.0 ([#3049](https://github.com/k0rdent/kcm/pull/3049)) by @dependabot
* **chore(deps)**: bump k8s.io/kubectl from 0.36.4 to 0.37.0 ([#3041](https://github.com/k0rdent/kcm/pull/3041)) by @dependabot
* **chore(deps)**: bump github.com/fluxcd/helm-controller/api ([#3048](https://github.com/k0rdent/kcm/pull/3048)) by @dependabot
* **chore(deps)**: bump github.com/onsi/gomega from 1.42.1 to 1.43.0 ([#3043](https://github.com/k0rdent/kcm/pull/3043)) by @dependabot
* **chore(deps)**: bump sigs.k8s.io/cluster-api-operator from 0.28.0 to 0.29.0 ([#3039](https://github.com/k0rdent/kcm/pull/3039)) by @dependabot
* **chore(deps)**: bump k8s.io/apiserver from 0.36.3 to 0.36.4 ([#3031](https://github.com/k0rdent/kcm/pull/3031)) by @dependabot
* **chore(deps)**: bump k8s.io/kubectl from 0.36.3 to 0.36.4 ([#3030](https://github.com/k0rdent/kcm/pull/3030)) by @dependabot
* **chore(deps)**: bump github.com/stretchr/testify from 1.12.0 to 1.12.1 ([#3025](https://github.com/k0rdent/kcm/pull/3025)) by @dependabot
* **chore(deps)**: bump helm.sh/helm/v3 from 3.21.3 to 3.21.4 ([#3019](https://github.com/k0rdent/kcm/pull/3019)) by @dependabot
* **chore(deps)**: bump github.com/stretchr/testify from 1.11.1 to 1.12.0 ([#3020](https://github.com/k0rdent/kcm/pull/3020)) by @dependabot
* **chore(deps)**: bump golang.org/x/net from 0.57.0 to 0.58.0 ([#3016](https://github.com/k0rdent/kcm/pull/3016)) by @dependabot
* **chore(deps)**: bump github.com/onsi/ginkgo/v2 from 2.32.0 to 2.32.1 ([#3011](https://github.com/k0rdent/kcm/pull/3011)) by @dependabot
* **chore(deps)**: bump golang.org/x/text from 0.40.0 to 0.41.0 ([#3012](https://github.com/k0rdent/kcm/pull/3012)) by @dependabot
* **chore(bump)**: k0s to v1.36.3+k0s.2 ([#3006](https://github.com/k0rdent/kcm/pull/3006)) by @eromanova
* **chore(deps)**: bump github.com/google/cel-go from 0.30.0 to 0.31.0 ([#3005](https://github.com/k0rdent/kcm/pull/3005)) by @dependabot
* **chore(bump)**: gcp 0.14.7 and os 1.13.1 ([#3004](https://github.com/k0rdent/kcm/pull/3004)) by @eromanova
* **chore(deps)**: bump github.com/fluxcd/source-controller/api ([#2999](https://github.com/k0rdent/kcm/pull/2999)) by @dependabot
* **chore(deps)**: bump kubevirt.io/containerized-data-importer-api ([#3000](https://github.com/k0rdent/kcm/pull/3000)) by @dependabot
* **chore(bump)**: velero to v1.18.2 ([#2998](https://github.com/k0rdent/kcm/pull/2998)) by @eromanova
* **chore(bump)**: k0s to v1.36.3+k0s.1 ([#2995](https://github.com/k0rdent/kcm/pull/2995)) by @eromanova
* **chore(bump)**: k0smotron to v2.1.0 ([#2992](https://github.com/k0rdent/kcm/pull/2992)) by @eromanova
* **chore(deps)**: bump CAPA provider from 2.12.1 to 2.13.0 ([#2987](https://github.com/k0rdent/kcm/pull/2987)) by @zerospiel
* **chore(bump)**: gcp provider to v1.13.0 ([#2986](https://github.com/k0rdent/kcm/pull/2986)) by @eromanova
* **chore(deps)**: bump kubevirt.io/api from 1.8.4 to 1.9.0 ([#2982](https://github.com/k0rdent/kcm/pull/2982)) by @dependabot
* **chore(deps)**: bump github.com/cert-manager/cert-manager ([#2976](https://github.com/k0rdent/kcm/pull/2976)) by @dependabot
* **chore(deps)**: bump cert-manager chart from 1.21.0 to 1.21.1 ([#2979](https://github.com/k0rdent/kcm/pull/2979)) by @zerospiel
* **chore(deps)**: bump github.com/google/cel-go from 0.29.2 to 0.30.0 ([#2968](https://github.com/k0rdent/kcm/pull/2968)) by @dependabot
* **chore(deps)**: bump github.com/prometheus/client_golang ([#2961](https://github.com/k0rdent/kcm/pull/2961)) by @dependabot
* **chore(deps)**: bump k8s.io/kubectl from 0.36.2 to 0.36.3 ([#2957](https://github.com/k0rdent/kcm/pull/2957)) by @dependabot
* **chore(deps)**: bump github.com/fluxcd/helm-controller/api ([#2958](https://github.com/k0rdent/kcm/pull/2958)) by @dependabot
* **chore(deps)**: bump k8s.io/apiserver from 0.36.2 to 0.36.3 ([#2956](https://github.com/k0rdent/kcm/pull/2956)) by @dependabot
* **chore(deps)**: bump github.com/go-logr/logr from 1.4.3 to 1.4.4 ([#2945](https://github.com/k0rdent/kcm/pull/2945)) by @dependabot

---

### Other Changes (CI, Tests, Refactors)

* **test**: raise unit-test coverage across previously untested packages ([#3044](https://github.com/k0rdent/kcm/pull/3044)) by @valentynkhalin
* **refactor**: Remove unused internal/sveltos/status.go ([#3038](https://github.com/k0rdent/kcm/pull/3038)) by @wahabmk
* **chore(deps)**: bump mikefarah/yq in the actions-minor-patch group ([#3032](https://github.com/k0rdent/kcm/pull/3032)) by @dependabot
* **chore(deps)**: bump docker/setup-buildx-action ([#3026](https://github.com/k0rdent/kcm/pull/3026)) by @dependabot
* **refactor**: key the RESTMapper cache by digests and make its size configurable ([#3022](https://github.com/k0rdent/kcm/pull/3022)) by @srnbckr
* **test(e2e)**: do not set insecureRegistry when using public repo ([#2980](https://github.com/k0rdent/kcm/pull/2980)) by @eromanova
* **chore**: fix unquoted linter template issues ([#2978](https://github.com/k0rdent/kcm/pull/2978)) by @eromanova
* **chore(deps)**: bump docker/login-action in the actions-minor-patch group ([#2977](https://github.com/k0rdent/kcm/pull/2977)) by @dependabot
* **chore(deps)**: bump docker/login-action in the actions-minor-patch group ([#2972](https://github.com/k0rdent/kcm/pull/2972)) by @dependabot
* **chore(deps)**: bump actions/stale from 10.4.0 to 11.0.0 ([#2973](https://github.com/k0rdent/kcm/pull/2973)) by @dependabot
* **chore(deps)**: bump docker/login-action in the actions-minor-patch group ([#2962](https://github.com/k0rdent/kcm/pull/2962)) by @dependabot
* **chore(deps)**: bump docker/login-action in the actions-minor-patch group ([#2959](https://github.com/k0rdent/kcm/pull/2959)) by @dependabot
* **chore(deps)**: bump actions/checkout in the actions-minor-patch group ([#2941](https://github.com/k0rdent/kcm/pull/2941)) by @dependabot

---

## References

* [Compare KCM v1.11.0...v1.12.0](https://github.com/k0rdent/kcm/compare/v1.11.0...v1.12.0)
