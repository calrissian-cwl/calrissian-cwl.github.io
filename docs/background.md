## About Calrissian

Calrissian is a CWL implementation designed to run inside a Kubernetes cluster. Its goal is to be highly efficient and scalable, taking advantage of high capacity clusters to run many steps in parallel.

It leverages `cwltool` to manage the execution of CWL workflows and triggers pods instead of containers. Here's a brief history of the Calrissian CWL runner:

## History and Development of Calrissian CWL Runner

### Inception and Need

**Early Days of CWL:**

The Common Workflow Language (CWL) was developed to provide a standardized way to describe workflows across various computing environments. CWL aimed to solve the problem of workflow portability and reproducibility in computational research.

**Containerization:**

As CWL gained traction, the need to run these workflows in containerized environments became apparent. Containers, with tools like Docker or podman, allowed for reproducible environments. However, traditional CWL runners were not optimized for containerized or cloud-native execution.


### Development of Calrissian

**Initial Development:** 

Calrissian was created to fill the gap in running CWL workflows efficiently in Kubernetes clusters. The initial version was developed by the Duke Center for Genomic and Computational Biology in collaboration with other contributors from the CWL community.

**Open Source:**

Calrissian is open-source, and its development has been community-driven, with contributions from various developers and researchers in the CWL community.

**Features and Evolution:**

* Kubernetes Integration: Calrissian leverages Kubernetes for resource allocation, scaling, and managing workflow tasks. It automatically schedules jobs, handles retries, and scales up resources as needed.
* CWL Compliance: Calrissian seeks to adhere strictly to the CWL standards, ensuring that workflows defined in CWL can be executed without modification.
* Performance Optimizations: Over time, Calrissian has seen several performance improvements, particularly in how it manages containerized jobs, handles data inputs/outputs, and integrates with Kubernetes features.

**Community Adoption:**

 Calrissian has been adopted by various research institutions and companies that rely on CWL for their workflow management, especially in bioinformatics, computational biology and Earth Observation.

**Current Status:**

* Active Development: As of now, Calrissian continues to be actively developed and maintained. It is widely used in the scientific community, particularly for workflows that require cloud-native execution.
* Growing Ecosystem: The ecosystem around CWL, including Calrissian, continues to grow, with more tools and integrations being developed to make workflow execution more seamless and efficient.

## Calrissian role

Calrissian CWL runner is a key tool for running CWL workflows in containerized environments like Kubernetes. Its development was driven by the need for a scalable, cloud-native solution for CWL workflows, and it has since become an essential tool in the CWL ecosystem.

The ongoing contributions from the open-source community ensure that Calrissian remains a cutting-edge solution for managing CWL workflows in modern computational environments.