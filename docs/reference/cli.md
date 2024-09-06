# Calrissian Command-Line Tool Reference

`calrissian` command-line tool reuses and extends the `cwltool` command-line tool interface.

## cwltool mandatory cli options

`calrissian` command-line tool reuses the `cwltool` command-line tool interface.

These options described below are mandatory when using `calrissian`.

### --tmp-outdir-prefix

`--tmp-outdir-prefix` sets the prefix for temporary output directories.

When a CWL tool or workflow is executed, `calrissian` creates temporary directories to store intermediate output files. 

By default, these directories are created in the system's temporary directory (e.g., `/tmp/`).

The `--tmp-outdir-prefix` option allows you to specify a custom prefix for these temporary directories in the ReadWriteMany volume, which can be useful for organizing or controlling the location of these directories.


### --outdir

`--outdir` specifies the output directory for final results.

When `calrissian` completes the execution of a workflow or tool, the final output files are stored in a designated directory. 

The `--outdir` option allows you to define the location of this output directory in the ReadWriteMany volume.

## calrissian extended cli options

`calrissian` command-line tool extends the `cwltool` command-line tool interface.

These options are described below.

### --max-ram

`--max-ram` sets the maximum amount of RAM to use, e.g., 1048576, 512Mi, or 2G. Follows Kubernetes (k8s) resource conventions.

Calrissian spawns pods until the max RAM value is reached. Remaining pods remain pending until resources are freed or procured.

### --max-cores

`--max-cores` sets the maximum number of CPU cores to use.

This option limits the number of CPU cores available for the pods. If the specified number of cores is exceeded, additional pods will be queued until resources are available.

### --max-gpus

`--max-gpus` sets the maximum number of GPU cores to use.

This option is optional and, if provided, limits the GPU resources available for the pods. If the specified number of GPU cores is exceeded, additional pods will be queued until resources are available.

### --pod-labels

`--pod-labels` allows the user to specify a YAML file containing labels to add to the Pods submitted.

These labels can be used to categorize, organize, or manage the Pods in the Kubernetes environment.

### --pod-env-vars

`--pod-env-vars` allows the user to specify a YAML file containing environment variables to add at runtime to the Pods submitted.

This is useful for injecting configuration values or secrets into the pod environments.

### --pod-nodeselectors

`--pod-nodeselectors` allows the user to specify a YAML file containing node selectors to add to the Pods submitted.

Node selectors constrain the Pods to run only on specific nodes within the Kubernetes cluster that match the specified criteria.

### --pod-serviceaccount

`--pod-serviceaccount` sets the Service Account to use for pods management.

The Service Account defines the permissions and access controls for the Pods running in the Kubernetes cluster.

### --usage-report

`--usage-report` specifies the output JSON file name to record resource usage.

This file contains details on the resource utilization (CPU, RAM, GPU, etc.) of the Pods during execution, which can be used for monitoring and optimization purposes.

### --stdout

`--stdout` specifies the output file name to tee standard output (CWL output object).

This option captures the standard output of the tool and saves it to the specified file, in addition to displaying it in the console.

### --stderr

`--stderr` specifies the output file name to tee standard error to (includes tool logs).

This option captures the standard error output, including logs, and saves it to the specified file, in addition to displaying it in the console.

### --tool-logs-basepath

`--tool-logs-basepath` sets the base path for saving the tool logs.

This option defines the directory where the tool's log files will be stored. It helps in organizing logs by location.

### --conf

`--conf` allows the user to define the default values for the CLI arguments.

This option can be used multiple times to override or set default values for the various command-line options, simplifying repeated use cases.
