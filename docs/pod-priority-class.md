# Calrissian spawned pod priority class

See the Kubernetes official documentation on how to define a [PriorityClass](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#priorityclass).

Calrissian spawned pods' priority class can be defined using the CLI option `--pod-priority-class`.

Example:

```console
calrissian --pod-priority-class high-priority
```
