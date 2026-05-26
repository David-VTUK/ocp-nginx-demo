# Advanced Cluster Management Application example with `Kustomize`

This repo is an example of how to leverage the RHACM application deployment (Push Model) in conjunction with `kustomize` to apply environment-specific modifications to how the application is deployed.

In `nginx-kustomize` there is:

* A `base` folder containing manifests with a default, baseline configuration
* A `overlays` folder presenting how this application needs to be deployed depending on the environment type (`prod` and `dev`)

In `rhacm-manifests` there is a `demo.yaml` file containing all the RHACM specific API's needed to deploy this application, namely:

* `ManagedClusterSets` - Representing `Dev` and `Prod` environment (you'll need to add clusters into these groups after).
* `ManagedClusterSetBinding` - Giving OpenShift Gitops visibility of the clusters so it can push applications to them.
* `Placement` - The "glue" between clusters and apps, the scheduling engine.
* `GitOpsCluster` - The bridge that connects RHACM's cluster-management engine with Red Hat OpenShift GitOps.
* `ApplicationSet` - The ArgoCD `application` factory that generates `Application` objects specific to each environment type.

The `path` and `server` variables are rendered in real time, based on RHACM labels which denote which manifests get pushed to which clusters.

```yaml
    spec:
      project: default
      source:
        repoURL: https://github.com/David-VTUK/rhacm-application-kustomize-example.git
        targetRevision: main
        path: 'nginx-kustomize/overlays/{{index .metadata.labels "cluster.open-cluster-management.io/clusterset"}}'
      destination:
        namespace: nginx-demo
        server: '{{.server}}'
```