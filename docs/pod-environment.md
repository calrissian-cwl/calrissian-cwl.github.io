# Calrissian spawned pod environment

Calrissian provides several methods to set the spawned pod environment variables.

## Using ConfigMaps

The CLI option `--env-from-configmap` allows settings the spawned pod environment from a ConfigMap. This CLI option can be provided multiple times.

If you have a ConfigMap with: 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

And use:

```console
calrissian --env-from-configmap special-config
```

The Calrissian spawned pods will have `SPECIAL_LEVEL: very` and `SPECIAL_TYPE: charm` amongst the environment variables.

## Using Secrets

The CLI option `--env-from-secret` allows settings the spawned pod envionment from a Secret. This CLI option can be provided multiple times.

## Using a local file

The CLI option `--pod-env-vars` allows the user to specify a YAML file containing environment variables to add at runtime to the Pods submitted.

Example:

```console
calrissian --pod-env-vars a-local-file
```

Where the local file `a-local-file` contains:

```yaml
SPECIAL_LEVEL: very
SPECIAL_TYPE: charm 
```