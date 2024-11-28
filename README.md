# Argilla Helm Chart

This Helm chart deploys Argilla Server on Kubernetes.

## Prerequisites

- Kubernetes 1.12+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure (if persistence is enabled)

## To install it locally

### Prepare minikube cluster
Create minikube cluster:
``` bash
minikube start --profile=argilla \
    --nodes=3 \
    --memory=4096 \
    --cpus=2 \
    --disk-size 2000mb
```
Enable ingress addon:
``` bash
minikube addons enable ingress -p argilla
```
Add needed helm repos:
```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add elastic https://helm.elastic.co
helm repo update
```
As we want to use the operator to manage our Elasticsearch cluster, we need to install the ECK operator.
```bash
helm install elastic-operator elastic/eck-operator -n elastic-system --create-namespace
```
The elastic-operator pod should be in the `Running` state.
```bash
kubectl get pods -n elastic-system
```
Run dependency build to ensure all dependency are installed:
``` bash
helm dependency build
```

### Installing the Argilla via chart

After adding the repository, you can install the chart with the release name `my-argilla-server`:
```bash
helm install my-argilla-server .
```

Check the status of the pods:
```bash
kubectl get pods -w
```
All the pods should be in the `Running` state.

### Accessing Argilla

In a different terminal window, run the following command to access Argilla:
```bash
kubectl port-forward svc/my-argilla-server 6900
```
Argilla will be accessible at http://localhost:6900.

## Execute integration tests

Set the following environment variables:
```bash
export ARGILLA_API_URL=http://localhost:6900
export ARGILLA_API_KEY=argilla.apikey
```

Run the following command to execute the integration tests:
```bash
pytest tests/integration
```

## Uninstalling the Chart

To uninstall/delete the `my-argilla-server` deployment:

```bash
helm delete my-argilla-server
```

This command removes all the Kubernetes components associated with the chart and deletes the release.

## Running Unit Tests

This chart includes unit tests using the helm-unittest plugin. To run these tests, follow these steps:

### Installing helm-unittest plugin

1. Install the helm-unittest plugin:

```bash
helm plugin install https://github.com/helm-unittest/helm-unittest.git
```

2. Verify the installation:

```bash
helm unittest --help
```

### Running the tests

To execute the unit tests for this chart, run the following command from the root of the chart directory:

```bash
helm unittest examples/deployments/k8s/argilla-chart
```

This will run all the test files located in the `tests/` directory of the chart.

For more information on writing and running helm unit tests, please refer to the [helm-unittest documentation](https://github.com/helm-unittest/helm-unittest).
