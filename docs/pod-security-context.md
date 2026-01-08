# Calrissian spawned pod security context

Calrissian spawned pods' default container security context is

* `readOnlyRootFilesystem: true` by default for all step containers, with default cli option `--no-read-only` to restore a writable root if needed
* `allowPrivilegeEscalation: false` set explicitly on all spwaned pods
* `privileged: false` set explicitly on all step containers. Note: this is the default in Kubernetes, but it is declared for clarity

Example: 

To set `readOnlyRootFilesystem` to `false` use:

```console
calrissian --no-read-only
```