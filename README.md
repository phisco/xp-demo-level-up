# Implementing a Composition with functions Demo

## Create a new control plane

```
kind create cluster
helm install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace
```

## Setup

```
# first generate GCP creds file with https://docs.crossplane.io/latest/getting-started/provider-gcp/#generate-a-gcp-service-account-json-file
kubectl create secret generic gcp-secret -n crossplane-system --from-file=creds=./gcp-credentials.json

kubectl apply -f setup
```

## Rendering
```
crossplane beta render examples/xr-dev.yaml setup/composition.yaml setup/functions.yaml --extra-resources=setup/environmentConfigs.yam

crossplane beta render examples/xr-dev-with-db.yaml setup/composition.yaml setup/functions.yaml --extra-resources=setup/environmentConfigs.yam

crossplane beta render examples/xr-prod.yaml setup/composition.yaml setup/functions.yaml --extra-resources=setup/environmentConfigs.yam

crossplane beta render examples/xr-prod-bigger.yaml setup/composition.yaml setup/functions.yaml --extra-resources=setup/environmentConfigs.yam
```

## Apply

```
kubectl apply -f examples/xr-dev-with-db.yaml

crossplane beta trace xacmedatabases.acme.com acme-db-dev -o wide
```
