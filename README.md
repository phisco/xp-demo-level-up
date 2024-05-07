# Implementing a Composition with functions Demo

We want to define a new resource `AcmeDatabase`(XRC), `XAcmeDatabase`(XR), to
create CloudSQL PG instances (`DatabaseInstance`).

We have two environments, `dev` and `prod`, for both of them we have some
environment-specific configurations.

We want to let users specify wether they want to have a `Database` created too,
but if specified, the `Database` can only be created once the DatabaseInstance is
ready.

## Developing it

- We define the [Provider](./setup/providers.yaml) we need.
- We define our [XRD](./setup/xrd.yaml).
- We define our environment specific configurations as [EnvironmentConfigs](./setup/environmentConfigs.yaml).
- We define our [Composition](./setup/composition.yaml) and all the [functions](./setup/functions.yaml) required.

### Composition logic

In pseudocode the above Composition logic could look like this:
```golang
# function-environment-configs
environment := getEnvironmentConfig(xr.spec.environment)

# function-patch-and-transform
desiredState := getDesiredResources(xr, environment)

# function-sequencer
desiredState = filterDatabaseIfDatabaseInstanceNotReady(desiredState)

# function-cel-filter
desiredState = filterDatabaseIfNotNeeded(xr, desiredState)

return desiredState
```

### Checking it locally

We can write a few examples:
- [xr-dev.yaml](./examples/xr-dev.yaml): a DatabaseInstance with the `dev` environment configuration.
- [xr-dev-with-db.yaml](./examples/xr-dev-with-db.yaml): a DatabaseInstance and a Database with the `dev` environment configuration.
- [xr-prod.yaml](./examples/xr-prod.yaml): a Database with the `prod` environment configuration.
- [xr-prod-bigger.yaml](./examples/xr-prod-bigger.yaml): a Database with a larger disk than the default, 100GB.

We can then render them all locally to see everything works as expected:
```
crossplane beta render examples/xr-dev.yaml setup/composition.yaml setup/functions.yaml --extra-resources=setup/environmentConfigs.yam

crossplane beta render examples/xr-dev-with-db.yaml setup/composition.yaml setup/functions.yaml --extra-resources=setup/environmentConfigs.yam

crossplane beta render examples/xr-prod.yaml setup/composition.yaml setup/functions.yaml --extra-resources=setup/environmentConfigs.yam

crossplane beta render examples/xr-prod-bigger.yaml setup/composition.yaml setup/functions.yaml --extra-resources=setup/environmentConfigs.yam
```

Have a look at [crossplane beta
validate](https://docs.crossplane.io/latest/cli/command-reference/#beta-validate)
to validate the rendered resources against their schema.

## Running it

### Create a new control plane

```
kind create cluster
helm install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace
```

### Setup

We'll first need to create a Secret with the GCP credentials, see the
[docs](https://docs.crossplane.io/latest/getting-started/provider-gcp/#generate-a-gcp-service-account-json-file)
for more details on how to get the token in the first place.

```
kubectl create secret generic gcp-secret -n crossplane-system --from-file=creds=../gcp-credentials.json
```

Now we can apply all the requirements:
```
kubectl apply -f setup
```

### Apply

Once all the requirements are successfully installed we can apply one of our
examples and check it works as intended.

```
kubectl apply -f examples/xr-dev-with-db.yaml

crossplane beta trace xacmedatabases.acme.com acme-db-dev -o wide
```
