#### Quotas {#container-registry-quotas}

#|
|| Type of limit | Value ||
|| Maximum number of [registries](../container-registry/concepts/registry.md) per [cloud](../resource-manager/concepts/resources-hierarchy.md#cloud) 
`container-registry.registries.count` | 10 ||
|| Number of concurrent [Docker image](../container-registry/concepts/docker-image.md) [scans](../container-registry/concepts/vulnerability-scanner.md) running in a single cloud 
`container-registry.parallelScans.count` | 10 ||
|#


#### Limits {#container-registry-limits}

Type of limit | Value
--- | ---
Maximum size of a single layer of a Docker image | 150 GB
Maximum number of layers in a Docker image being scanned | 127
Maximum number of Docker images in a single [repository](../container-registry/concepts/repository.md) that can be checked by a trial run of the [Docker image lifecycle policy](../container-registry/concepts/lifecycle-policy.md) | 50 000
Maximum size of a Docker image to scan | 2 GB
Maximum number of Docker images to [scan](../container-registry/operations/scanning-docker-image.md) per request | 10 000
