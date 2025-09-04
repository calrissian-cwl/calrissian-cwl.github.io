
## Environment

Calrissian process must run in a pod in a kubernetes cluster.

This process must have an environment variable name **CALRISSIAN_POD_NAME** used by Calrissian to identify the Pod in which it is running. 

Calrissian uses this information to list the volumes mounted on the current Pod and to "re-mount" these volumes on the Pods that will be created for running individual CWL tools. 

This ensures that the tools have access to the necessary files and directories specified in the CWL workloads.

## Scalability / Resource Requirements

Calrissian is designed to issue tasks in parallel if they are independent, and thanks to Kubernetes, should be able to run very large parallel workloads.

When running calrissian, you must provide a limit the the number of CPU cores (`--max-cores`), RAM megabytes (`--max-ram`) and GPU (`--max-gpu`) to use concurrently. Calrissian will use CWL ResourceRequirements to track usage and stay within the limits provided. 

It is highly recommended using accurate ResourceRequirements in the workloads, so that they can be scheduled efficiently and are less likely to be terminated or refused by the cluster.

### CPU

The `--max-cores` option represents the total number of CPU cores allocated across all the Pods managed by Calrissian. 
The value is specified as an integer. 
It sets the maximum number of CPU cores that Calrissian can use for the execution of concurrent workflows on that pool of resources.

How it works:

* Calrissian tracks the total number of CPU cores being utilized by all active Pods.
* Pods are scheduled to run on the cluster as long as the cumulative CPU usage remains below the specified `--max-cores` limit.
* If starting a new Pod would cause the total CPU usage to exceed this limit, the Pod will remain in a pending state until sufficient CPU resources are freed by the completion or termination of other Pods.

This option is crucial for managing CPU resources effectively within a Kubernetes environment, ensuring that the workflow does not overcommit CPU resources, which could lead to performance issues or throttling by the Kubernetes scheduler.

### Memory

The `--max-ram` option represents the total RAM allocated across all the Pods managed by Calrissian. 
The value  is specified in bytes (e.g., 1048576), mebibytes (e.g., 512Mi), or gigabytes (e.g., 2G), following Kubernetes (k8s) resource conventions.
It sets the maximum amount of RAM that Calrissian can use for the execution of concurrent workflows on that pool of resources.

How it works:

* Calrissian monitors the total RAM consumption of all the Pods it manages.
* Pods are created and scheduled to run on the cluster as long as the cumulative RAM usage remains below the specified `--max-ram` limit.
* If launching a new Pod would exceed this limit, the Pod remains in a pending state until sufficient resources are freed by the completion or termination of other Pods.

This mechanism ensures that a workflow adheres to the specified memory constraints, preventing over-allocation of resources that could lead to performance degradation or failures in a Kubernetes environment.

### GPU

The `--max-gpus` option represents the total number of GPU cores available for allocation across all Pods managed by Calrissian.
The value is specified as an integer.
It sets the maximum number of GPU cores that Calrissian can use for the execution of concurrent workflows on that pool of resources.

How it works:

* Calrissian monitors the total number of GPU cores in use by all active Pods.
* Pods that require GPU resources are scheduled to run on the cluster as long as the cumulative GPU usage remains below the specified `--max-gpus` limit.
* If launching a new Pod would exceed this GPU limit, the Pod remains in a pending state until sufficient GPU resources are freed by the completion or termination of other Pods.

This option is essential for workflows that involve GPU-intensive tasks, such as machine learning or scientific computations, ensuring that GPU resources are managed effectively and preventing over-allocation that could lead to resource contention or job failures.

## Resource consumption report

The `--usage-report` option specifies the path to a JSON file where Calrissian will record detailed resource usage metrics and job outcomes during the execution of a workflow. This report provides valuable insights into how resources were utilized and the performance of each task within the workflow.

**What it includes:**

* CPU Cores: The total number of CPU cores used by each Pod during its execution.
* RAM Usage: The amount of RAM consumed by each Pod throughout its lifecycle.
* GPU Usage (if applicable): The number of GPU cores utilized by Pods, relevant if the workflow involves GPU resources.
* Exit Codes: The exit codes returned by each Pod, indicating the success or failure of the tasks they performed.

**How it works:**

Throughout the workflow execution, Calrissian tracks the resource usage and exit codes of all Pods it manages.

Upon completion, Calrissian compiles this data into a JSON file at the location specified by the `--usage-report` option.

This JSON report can be used for post-execution analysis, helping users understand resource allocation, optimize future runs, and troubleshoot any issues that may have occurred.

This option is particularly useful for monitoring and optimizing workflows, as it provides a comprehensive overview of how resources were consumed during execution, as well as the success or failure of each task.

## CWL execution results

The `--stdout` option specifies the file where Calrissian will dump the standard output produced by the CWL (Common Workflow Language) execution, which typically includes the results or outputs generated by the workflow.

**How it works:**

During the execution of a CWL workflow, various tools and steps produce outputs. These outputs, often formatted as JSON, describe the results of the workflow, including any files, data, or other artifacts generated.

The `--stdout` option directs this output to a specified file, rather than displaying it directly in the terminal.

By capturing the output in a file, users can easily access and review the results after the workflow completes.


Benefits:

* Convenience: Storing output in a file allows for easy access and review, especially for large outputs that may be cumbersome to read in the terminal.
* Post-processing: The output file can be used for further processing, such as parsing the results with scripts, sharing the file with collaborators, or integrating with other tools.
* Record-keeping: The output file serves as a permanent record of the workflow results, which can be useful for documentation or reproducibility.

This option is essential for workflows where the output data is important for further analysis, sharing, or archiving, ensuring that the results are captured in a structured and accessible format.

## Calrissian execution log

The `--stderr` option specifies the file where Calrissian will log the standard error output, which includes detailed execution logs of the Calrissian process, as well as any error messages, warnings, or diagnostic information generated during the workflow execution.

**How it works:**

During the execution of a CWL (Common Workflow Language) workflow, Calrissian generates various log entries and error messages that are normally output to the terminal's standard error stream.

The `--stderr` option directs this standard error output to a specified file, ensuring that all relevant logs and error information are captured for review.

This log file contains critical information about the workflow execution, including any issues that occurred, steps performed, and resource usage details, making it an invaluable resource for debugging and auditing.

**Benefits:**

* Debugging: By capturing standard error output in a file, users can systematically review the logs to diagnose and resolve any issues that may have occurred during the workflow execution.
* Execution Trace: The log provides a step-by-step trace of the execution process, which is useful for understanding how the workflow was processed and where any potential bottlenecks or failures might have occurred.
* Audit Trail: The log file serves as an audit trail, documenting the workflow execution in detail, which can be important for compliance, reproducibility, or record-keeping.

This option is crucial for maintaining a clear and accessible record of the Calrissian execution process, enabling users to troubleshoot problems, monitor the workflow, and ensure that everything runs as expected.
