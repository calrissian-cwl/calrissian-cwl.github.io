Calrissian's nominal behaviour is to schedule pods per CWL steps while CWL tools may need distributed compute for array- or window-based processing. 

The Calrissian DaskGatewayRequirement extension provides a first-class way to provision and manage a transient Dask cluster that a CommandLineTool can talk to.

DaskGatewayRequirement extension lets a tool request a short-lived Dask cluster (via Dask Gateway) as part of step execution. 

Calrissian will:

* Bring up a Dask cluster prior to tool execution (init container),
* Surface scheduler connection info to the main container,
* Optionally run user-provided init and dispose scripts,
* Tear everything down reliably (including on failures).

The Calrissian DaskGatewayRequirement extension makes Dask usage declarative and portable at the CWL layer while keeping cluster lifecycle under Calrissian control. 

### DaskGatewayRequirement extension schema

Below the schema for the DaskGatewayRequirement extension:

```yaml
$base: https://calrissian-cwl.github.io/schema#
$namespaces:
  cwl: "https://w3id.org/cwl/cwl#"
$graph:
- $import: https://w3id.org/cwl/CommonWorkflowLanguage.yml

- name: DaskGatewayRequirement
  type: record
  extends: cwl:ProcessRequirement
  inVocab: false
  doc: "Indicates that a process requires a Dask cluster procured via [Dask Gateway](https://gateway.dask.org/) runtime."
  fields:
    class:
      type: 'string'
      doc: "Always 'DaskGatewayRequirement'"
      jsonldPredicate:
        "_id": "@type"
        "_type": "@vocab"
    workerCores:
      type:
        - 'int'
        - 'cwl:Expression'
      doc: |
        Number of cpu-cores available for a dask worker.
    workerCoresLimit:
      type:
        - 'int'
        - 'cwl:Expression'
      doc: |
        Maximum number of cpu-cores available for a dask worker.
    workerMemory:
      type:
        - 'string'
        - 'cwl:Expression'
      doc: |
        Maximum amount of memory available for a dask worker.
    clusterMaxCores:
      type:
        - 'int'
        - 'cwl:Expression'
      doc: |
        Maximum number of cores available for the dask cluster
    clusterMaxMemory:
      type:
        - 'string'
        - 'cwl:Expression'
      doc: |
        Maximum amount of memory available for the dask cluster
```

### CWL CommandlineTool example


```yaml
cwlVersion: v1.2

$namespaces:
  s: https://schema.org/
  calrissian: https://calrissian-cwl.github.io/schema#
schemas:
  - http://schema.org/version/9.0/schemaorg-current-http.rdf

class: CommandLineTool
id: wrs-coverage-tool
requirements:
    EnvVarRequirement:
    envDef: {}
    NetworkAccess:
    networkAccess: true
hints:
    DockerRequirement:
        dockerPull: docker.io/library/wrs-coverage
    calrissian:DaskGatewayRequirement:
        workerCores: 1
        workerCoresLimit: 1
        workerMemory: "1G"
        clusterMaxCores: 10
        clusterMaxMemory: "20G"
baseCommand: ["wrs-coverage"]
arguments: []
inputs:
    collection-id:
    type: string
    inputBinding:
        position: 1
        prefix: "--collection-id"
outputs:
    wrs-coverage-image:
    type: File
    outputBinding:
        glob: acq-by-wrs-tile.png
    wrs-coverage-parquet:
    type: File
    outputBinding:
        glob: acq-by-wrs-tile.parquet
```

### Additional CLI options

The Calrissian CLI options below allow defining the Dask Gateway configuration:

* `--dask-gateway-url`: defines the Dask Gateway URL. This is the Dask Gateway internal service URL.

Optional:

* `--dask-script-configmap`: name of an existing configmap with custom script for dask to override the default Dask cluster initialization. 