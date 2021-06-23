---
title: Enable Observbility Service
weight: 7
---

After the cluster manager is installed, you could enable the observability service in the hub cluster to monitor the health of your managed clusters.

<!-- spellchecker-disable -->

{{< toc >}}

<!-- spellchecker-enable -->

## Prerequisite

You must meet the following prerequisites to enable observability service:

* Ensure [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl) and [kustomize](https://kubernetes-sigs.github.io/kustomize/installation) are installed.

* Prepare one OKD cluster to function as the hub cluster.

* Ensure [docker 17.03+](https://docs.docker.com/get-started) is installed.

* Ensure [golang 1.15+](https://golang.org/doc/install) is installed.

* Ensure [operator-sdk 1.4.2+](https://github.com/operator-framework/operator-sdk) in installed.

* Ensure the open-cluster-management cluster manager is installed. See [Cluster Manager](/getting-started/core/cluster-manager) for more information.

* Ensure the `open-cluster-management` _klusterlet_ is installed. See [Klusterlet](/getting-started/core/register-cluster) for more information.

* Ensure the Application lifecycle management in installed. See [Application lifecycle management](/getting-started/integration/app-lifecycle.md) for more information.

## Install from source
Clone the `multicluster-observability-operator` repository.

```Shell
git clone https://github.com/open-cluster-management/multicluster-observability-operator
```

Create `open-cluster-management-observability` namespace:
```Shell
kubectl create namespace open-cluster-management-observability
```

Deploy `minio` as object storage:
```Shell
kubectl -n open-cluster-management-observability apply -f example/minio
```

Build multicluster-observability-operator image and push it to a public registry, such as:

```Shell
make -f Makefile.prow docker-build docker-push IMG=quay.io/<YOUR_USERNAME_IN_QUAY>/multicluster-observability-operator:latest
```

Deploy the multicluster-observability-operator to the hub cluster with the following commands:

```Shell
make -f Makefile.prow deploy IMG=quay.io/<YOUR_USERNAME_IN_QUAY>/multicluster-observability-operator:latest
```

Deploy the multicluster-observability-operator CR with the following commands:

```Shell
cat << EOF | kubectl apply -f -
apiVersion: observability.open-cluster-management.io/v1beta2
kind: MultiClusterObservability
metadata:
  annotations:
    mco-imageRepository: quay.io/open-cluster-management
    mco-imageTagSuffix: 2.3.0-SNAPSHOT-2021-06-23-07-28-33
  name: observability
spec:
  observabilityAddonSpec: {}
  storageConfig:
    metricObjectStorage:
      name: thanos-object-storage
      key: thanos.yaml
EOF
```
Note: you can get the latest SNAPSHOT tag from https://quay.io/repository/open-cluster-management/multicluster-observability-operator?tab=tags

## What is next

After a successful deployment, you can run the following commands to check if you have OCP cluster as a managed cluster.

```Shell
kubectl get managedcluster --show-labels
```
If there is `vendor=OpenShift` label exists in your managed cluster, you should be able to explore by thanos UI. If no, you can manually add this label. for example: `kubectl label <managed cluster name> vendor=OpenShift`

Expose the thanos query frontend by using route.

```Shell
cat << EOF | kubectl -n open-cluster-management-observability apply -f -
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: query-frontend
spec:
  port:
    targetPort: http
  wildcardPolicy: None
  to:
    kind: Service
    name: observability-thanos-query-frontend
EOF
```

Then you can access the route host from browser to explore.
